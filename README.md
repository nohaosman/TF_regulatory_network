# TF_regulatory_network

 #Read mapping

                                       Inference_of_TF_regulatory_networks/counting/map.sh
                                       
Reference genome used for mapping:

ftp://ftp.ensembl.org/pub/release-75/fasta/drosophila_melanogaster/dna/Drosophila_melanogaster.BDGP5.75.dna.toplevel.fa.gz

Annotation file used for mapping:

ftp://ftp.ensembl.org/pub/release-75/gtf/drosophila_melanogaster/Drosophila_melanogaster.BDGP5.75.gtf.gz

# Create count table
                                     Inference_of_TF_regulatory_networks/counting/count.sh
                                     
             output file : embryoCounts.tab (has 71 crosses .25 crosses were sequenced with two biological replicates and the remaining 46 crosses were sequenced without replicates.In total 96 samples . 
             
#Combination the two replicates of 25 crosses

                                embryoCounts_combination.pl 
                                
                   Input file :    embryoCounts.tab ( file has 96 samples and 156825 genes)
                   
                   Output file : embryoCountsresult_with_length.txt (file has 71 samples and 15682 genes plus last column represents the length of each gene from Flybase database)
                   
# Count Data normalization (RPKM)

                                                 RPKM_normalization.r
                                                 
                         Input file :   embryoCountsresult_with_length.txt
                         
                         Output file : rpkm59_exp50_genes_least_50_samples.tab (file has 59 samples and 7805 genes [Highly expressed genes in at least 50 samples)
                         
                         
# Expression levels of TFs which expressed in our data

                                                expression_level_of_TFs.r 
                                                
                          Input file 1 :    embryoCountsresult_with_length.txt
                          
                          Input file 2: TF_names397.txt ( has TFs names from FlyTF.org)
                          
                          Output file : tfselected.txt file (expression levels of TFs in 59 samples)
                          
# Correlation to get the  expression covariation  between highly expressed genes (7805 ) and all TFs expressed in our data.

                                                      Correlation.r 
                                                      
                             Input file 1:  rpkm59_exp50_genes_least_50_samples.tab
                             
                             Input file 2: tfselected.txt file
                             
                             Output file : cor_tf_gene.tab ( has expression covariation between expression levels of TFs (397) and highly expressed genes (7805 genes)
                             
                             
# Spearman's correlation  to get the  expression covariation  between highly expressed genes (7805 ) and 14 TFs of interest.    

                                             Expression_covariation_14_TFs.r  
                                             
                        Input file 1:  tfselected.txt file        
                         
                        Input file 2: gene_id_14_TFs (file has gene id of 14 TFs of interest)
                         
                       Input file 3: rpkm59_exp50_genes_least_50_samples.tab 
                         
                   Output file : expression_covariation_tf_14.txt  ( file has expression covariation between expression levels of 14 TFs of interest and highly expressed genes (7805 genes) .First column is genes_names which pulled out in file (genes_name_TF.txt) for next step)
                             
                             
#Pull out the positions of the genes which have correlated values between -1 and 1 with each TF of interest from gtf file

`awk '{if($3=="gene")print;}' Drosophila_melanogaster.BDGP5.75.gtf >result1.txt`

`sed -e 's/\"//g' result1.txt > result.txt`

`tail -n+2 genes_name_TF.txt |while read line; do grep $line result.txt | awk '{if($3=="gene") print;}' >>position_Genes.txt;done;`

# Accessible regions of genes which have correlation with each TF of interest in 5kb window
                             `5kb.pl`
                             
  Input file 1: position_Genes.txt file (positions of genes which have positive and negative correlation with each TF of interest from  Drosophila_melanogaster gtf file).
  
  Input file 2:correlated accessible regions file ( the accessible regions of Drosophila_melanogaster developmental stage which we are interested in from earlier study).
  
   Output file: position_accessible_regions_genes_TF.txt file ( positions of accessible regions of genes which have positive and negative correlation with each TF).

#Pull out the fasta sequence for  accessible regions of genes which have correlation with each TF of interest  from reference genome
                            `fastasequence.pl`

    Input file 1: Drosophila_melanogaster.BDGP5.75.dna.toplevel.fa (Fasta sequence of reference genome)
    
    Inputfile 2: position_accessible_regions_genes_TF.txt file
    
    Output file : fasta_sequence_accessible_regions_genes_TF.txt  (Fasta sequence of accessible regions of genes have correlation with each TF)


#PATSER program command to get the scores (strength binding) and numbers of binding sites in accessible regions of genes have correlation with PWM of each TF of interest from Fly Factor Survey.
`patser-v3e -A a:t 0.53 c:g 0.47 -f fasta_sequence_accessible_regions_genes_TF.txt  -m PWM of each TF of interest -c -li > TF_results_patser_scores.txt`

#Filtering patser output data

 Patser is run using the "-li" option which calculates cutoff binding score based on PWM's information content For each TF.

 
  A: Determine the top cutoffs 10% and 5 %  of binding scores for each TF.
  
  
                                    10_5_tail_score_distribution.r
  
                                Input file : TF_results_patser_scores.txt ((output file of patser)
                                
                                Output file : TF_score_distribution_plot.png (plot of score distribution with 10% and 5% tail values)
           
    B: Keep the genes have binding scores in top 10% and 5 % cutoffs for downstream analysis.
    
                             filteration_genes_5%_10%.pl
                             
                          Input file :  TF_results_patser_scores.txt
                          
                          Output file : TF_results_patser_scores_5_10.txt ( file has genes which in 10 % and 5 % tail cutoffs)


# Analysis after PATSER

A)  Count the numbers of binding sites for each gene in 10 and 5 % tail of score distribution with each TF motif (PWM) of interest ----output file contains the number of binding sites for each gene with TF of interest

                                     `count.pl`
                                     
     Input file (TF_results_patser_scores_5_10.txt) :  names and scores of the genes in 10% and 5% tail of score distribution
      
      Output file (number_binding_sites_TF.txt) : name and numbers of each gene in 10% and 5% tail of score distribution
                             


B) Calculate the average of  binding score for binding sites of each gene in 10 and 5 % tail of score distribution with  TF motif (PWM)of interest --output file contains the average of strength binding for each gene with each TF of interest.  

                                             `average_binding_score.pl`
                     
        Input file (TF_results_patser_scores_5_10.txt) :  names and scores of the genes in 10% and 5% tail of score distribution
        
        Output file (TF.avg.txt) : name and average strength binding of each gene in 10% and 5% tail of score distribution

C) Get genes_name,number of binding sites and average of binding sore for each gene in 10 and 5% tail of score distribution with each TF motif (PWM) of interest -- output file has number of binding sites and average of strength binding for each gene with each TF of interest.

                                     `countnumber_bindingscore.pl`
                                     
              Input file 1: number_binding_sites_TF.txt
              
              Input file 2: TF.avg.txt
              
              Output file : genes_names_count_average_tf.txt : name , number_binding_sites and average_binding of each gene in 10% and 5% tail of score distribution

D) Get genes_name , number of binding sites , average of binding sites , co-variation of each gene in 10% and 5% tail of score distribution with each TF motif (PWM) of interest.

                              `genes_numbers_score_expression.pl`
                              
                   Input file 1 : genes_names_count_average_tf.txt 
                   
                  Input file 2 : expression_covariation_tf_14.txt (expression covariation of each TF of interest with 7805 genes).
                  
                  Output file : genes_names_count_average_expression_covariation_tf.txt (has number_binding_sites , average_strength_binding, expression_covariation for each gene with each TF of interest)

E)  Keep the genes which have more than two binding sites then divide those genes into genes have positive and negative covariation with each TF of interest 

                                    `more_two_binding_sites_negative_positive.pl` 
                                    
                       Input file 1 : genes_names_count_average_expression_covariation_tf.txt
                       
                       Output file 1: negative.txt ( genes have more than two binding sites and negative covariation with TF)
                       
                       Output file 2 : positive.txt (genes have more than two binding sites and positive covariation with TF)

F)  Spearman correlations is done by R for positive.txt and negative.txt  to estimate the correlation between the number of binding sites and expressioncovariation as well as between average strength binding and expressioncovariation of each TF of interest.
            
                           # R script (positive.r)
                           
                           #  R script (negative.r)






  
 








 


 


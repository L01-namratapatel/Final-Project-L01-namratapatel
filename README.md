# Final-Project-L01-namratapatel
This will uncompress the proteosomes:
    gunzip proteomes/*.gz 
cat  proteomes/* > allprotein.fas #this will put all of the proteins into one single file
makeblastdb -in allprotein.fas -parse_seqids -dbtype prot #this will make a BLAST database
ncbi-acc-download -F fasta -m protein XP_032239066.1 #this will download the protein as a query sequence
blastp -db allprotein.fas -query XP_032239066.1.fa -outfmt 0 -max_hsps 1 -out rho.blastp.typical.out #this will perform a blast search using the query protein
awk '{if ($6<0.00000000000001)print $1 }' rho.blastp.detail.out > rho.blastp.detail.filtered.out #this will filter the output file
seqkit grep --pattern-file rho.blastp.detail.filtered.out allprotein.fas > rho.blastp.detail.filtered.fas #this will help obtain the gene family sequences and align it
muscle -in rho.blastp.detail.filtered.fas -out rho.blastp.detail.filtered.aligned.fas #this will perform a global multiple sequence alignment
t_coffee -other_pg seq_reformat -in rho.blastp.detail.filtered.aligned.fas -action +rm_gap 50 -out rho.blastp.detail.filtered.aligned.r50.fas #this will give some statistics aboout the alignment
echo "(((((Bbelcheri,Hsapiens),Aplanci),Myessoensis)),((Hvulgaris,(Dgigantea,Nvectensis))));" > species.tre #this will help make the species tree 
iqtree -s rho.blastp.detail.filtered.aligned_.fas -nt 2 #
java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s toyspecies.tre -g toygenetree.tre --reconcile --speciestag prefix --savepng --events #this will help find the maximum likelihood tree estimate
python2.7 ~/tools/recPhyloXML/python/NOTUNGtoRecPhyloXML.py -g rho.genes.midpoint.ufboot.reconciled --include.species #this will create a RecPhyloXML object file
thirdkind -f rho.blastp.detail.filtered.aligned_.fas.treefile.genes.tre.reconciled.xml -o  rho.blastp.detail.filtered.aligned_.fas.treefile.genes.tre.reconciled.svg #this will view the gene tree reconciliation of the species tree
java -jar ~/tools/Notung-3.0-beta/Notung-3.0-beta.jar -s speciesTreeBilateriaCnidaria.tre -g rho.genes.midpoint.ufboot --reconcile --speciestag prefix --savepng --events #this will perform the reconciliation of the rho gene
iqtree -s rho.blastp.detail.filtered.aligned_.fas -bb 1000 -nt 2 -m VT+F+R4 -t rho.blastp.detail.filtered.aligned_.fas.midpoint.treefile -pre rho.genes.ufboot #this will calculate the bootstrap support and perform the tree search
gotree reroot midpoint -i rho.genes.ufboot.treefile -o rho.genes.midpoint.ufboot #this will re-root the optimal tree 
nw_display -s  rho.blastp.detail.filtered.aligned_.fas.midpoint.treefile -w 1000 -b 'opacity:0' >  rho.blastp.detail.filtered.aligned_.fas.midpoint.treefile.svg #this will make a graphics file with the bootstrap support from the printed tree
mkdir ~/labs/lab8-L01-namratapatel/myfamily #this will make a separate directory for the output
cd ~/labs/final-project-L01-namratapatel/myfamily #this opens up a separate folder
iprscan5   --email namrata.patel@stonybrook.edu  --multifasta --useSeqId --sequence   rho.blastp.detailed.filtered.fas #this will send the sequences to the inteproscan servers where it can be analyzed
cat ~/labs/final-project-L01-namratapatel/myfamily/*.tsv.tsv > ~/labs/final-project-L01-namratapatel/rho.domains.all.tsv #this will join the files into one single file
grep Pfam ~/labs/final-project-L01-namratapatel/rho.domains.all.tsv >  ~/labs/final-project-L01-namratapatel/rho.domains.pfam.tsv #this will filter the files
awk 'BEGIN{FS="\t"} {print $1"\t"$3"\t"$7"@"$8"@"$5}' ~/labs/lab8-L01-namratapatel/rho.domains.pfam.tsv | datamash -sW --group=1,2 collapse 3 | sed 's/,/\t/g' | sed 's/@/,/g' > ~/labs/lab8-L01-namratapatel/rho.domains.pfam.evol.tsv #this will fix the output up by re-arranging the interproscan output

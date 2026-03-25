# germline-variant-pipeline-dfins
Iremos montar desde o início um **pipeline para análise de variantes germinativas**, desde o momento em que o sequenciador nos fornece o VCF das amostras, e temos que baixar e alinhar com o genoma de referência, até anotação das variantes candidatas.
# **PIPELINE DE ANÁLISE DE VARIANTES GERMINATIVAS**
___
# **PREPAROS INICIAIS**
**1. Conectar e sincronizar com o seu Drive**

Importante, para não perder informações durante o processo.

```bash
from google.colab import drive
drive.mount('/content/drive', force_remount=True)
```
___
**2. Criação de estrutura de pastas**

Vamos neste momento criar um diretório no drive para salvar o nosso progresso executando as etapas do nosso pipeline.

**PASSO OPCIONAL**. Caso você já tenha criado as pastas, não é necessário rodar esta célula.

```bash
%%bash
mkdir /content/drive/MyDrive/PGBIOAGMAV

# comando_para_criar_diretorio /content/drive/MyDrive/PGBIOAGMAV
# mkdir cria um novo diretório (pasta) / e dar o path completo
```

**2.1 - Criar variável  + pastas**

Criar uma variável contendo o path (caminho) para a minha pasta de interesse para não precisar ficar repetindo ela sempre;

E além disso, criar pastas para organizar melhor os dados dentro do diretório (PGBIOAGMAV).

```bash
%%bash

MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

mkdir $MeuDrive/Dados
mkdir $MeuDrive/Dados/fastq
mkdir $MeuDrive/Dados/bam
mkdir $MeuDrive/Dados/vcf
```

**2.2 - Baixar os fastqs do sequenciador e colocá-los na pasta do drive *"fastq"* criada.**

Esta parte pode ser feita de forma manual, sem código. Baixar os arquivos e arrastá-los para a pasta.
___

**3. Baixar o FastQC**

Para ver qualidade dos arquivos Fastqs

```bash
%%bash

apt-get update
apt-get install -y fastqc
```
___
# **PREPARAÇÃO DO GENOMA DE REFERÊNCIA**
**4. Baixar ferramentas**

Utilizar o código `apt-get install -y NOME_DA_FERRAMENTA` para baixar.

Sendo elas:

*   BWA = (Burrows-Wheeler Aligner). Ferramenta para **alinhamento** de sequências de DNA contra genoma de referência.
*   samtools = **Conjunto de utilitários** para manipulação de arquivos SAM/BAM e indexação de FASTA.
*   bcftools =
*   vcftools =
*   bedtools =

| Ferramenta | Função |
|------------|--------|
| **samtools** | Manipulação BAM/VCF |
| **bcftools** | Processamento VCF |
| **vcftools** | Análise e filtragem VCF |

```bash
%%bash

apt-get update -qq
apt-get install -y bwa samtools bcftools vcftools bedtools
# apt-get install -y NOME_DA_FERRAMENTA
```
___

**4.1 - Baixar ferramentas com arquivo em formato de URL (GATK e, PICARD e ANNOVAR)**

 Alguns programas vêm prontos para uso, sem a necessidade de instalação. Esses programas são distribuídos como **executáveis** ou pacotes que, após baixados e extraídos, já podem ser utilizados imediatamente.

 Diferentemente das outras, essas ferramentas possuem os seus respectivos arquivos em forma de URL, logo, a linha de comando usada para baixar é: `wget -q LINK_DA_FERRAMENTA`.


*   *GATK* =
*   *PICARD* =
*   *ANNOVAR* =

```bash
  %%bash

#Baixar GATK

wget -q https://github.com/broadinstitute/gatk/releases/download/4.1.8.1/gatk-4.1.8.1.zip
#É uma URL do arquivo. wget para fazer o download desse URL /  caso queira pegar mais atualizado, pesquisar no google, e colocar URL novo no lugar do antigo.

unzip -q gatk-4.1.8.1.zip
#Só para tirar do zip o arquivo que foi baixado

rm gatk-4.1.8.1.zip
# para apagar a pasta zipada e ficar só com a unzipada

#=-q - só para não poluir a tela (se quiser saber o que é um simbolo, colocar simbolo -- help / exemplo: -q --help)
#depois de baixar, a pasta gatk será criada (lembrando que ele não está sendo salvo no drive. Se for dado refresh, no caso do colab, teriamos que baixar o gatk e o picar. Se fosse no bash, instala uma vez e tá pra sempre), o executável será um arquivo dentro dele chamado gatk

```

```bash

%%bash

#Baixar PICARD

wget https://github.com/broadinstitute/picard/releases/download/2.24.2/picard.jar
#Não é zipado, logo, não precisa das duas outras etapas acima
# por não ter o -q, gera todas essas mensagems quando roda.
```

```bash

%%bash

# Baixar ANNOVAR

wget -q http://www.openbioinformatics.org/annovar/download/0wgxR2rIVP/annovar.latest.tar.gz
tar -xzf annovar.latest.tar.gz
```
___ 

**5. Baixar bancos de dados**


*   *RefGene* =
*   *gnomAD* =
*   *Revel* =
*   *Clinvar* =

```bash

%%bash

NÃO DEFINI GENOMA!!! DEFINIR...
echo "   📚 Baixando refGene (genes de referência)..."
perl annovar/annotate_variation.pl -buildver ${GENOMA} -downdb \
  -webfrom annovar refGene annovar/humandb

echo "   👥 Baixando gnomAD (frequências populacionais)..."
perl annovar/annotate_variation.pl -buildver ${GENOMA} -downdb \
  -webfrom annovar gnomad_exome annovar/humandb

echo "   🔬 Baixando REVEL (predições de patogenicidade)..."
perl annovar/annotate_variation.pl -buildver ${GENOMA} -downdb \
  -webfrom annovar revel annovar/humandb

echo "   🏥 Baixando ClinVar (significado clínico)..."
perl annovar/annotate_variation.pl -buildver ${GENOMA} -downdb \
  -webfrom annovar clinvar_20200316 annovar/humandb

echo ""
echo "✅ bancos de dados instalados com sucesso!"

```
___

**6. Criar pastas para armazenamento do genoma de referência**

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

# CRIAR_PASTA NOME_PASTA
mkdir $MeuDrive/referencia_tcc
mkdir $MeuDrive/referencia_tcc/hg19_tcc
mkdir $MeuDrive/referencia_tcc/hg38_tcc
```
___

**7. Baixar o `genoma Hg19` e criar um `.fasta (hg19_tcc.fasta)` dele, e colocar dentro do arquivo `hg19_tcc`**

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"


curl -s https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr1.fa.gz | \
   gunzip -c > $MeuDrive/referencia_tcc/hg19_tcc/hg19_tcc.fasta
```

**7.1 - Explorando o formato `.fasta`**

A linha de código `head -n 10 CAMINHO_DO_ARQUIVO` mostra as 10 primeiras linhas do arquivo em questão.

Obs: é possível variar o número de linhas mostradas a partir da mudança do número (pode colocar -n 20, 30, 40...).

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

head -n 10  $MeuDrive/referencia_tcc/hg19_tcc/hg19_tcc.fasta
```

___

**8. Fazendo a indexação com o BWA**

```bash

%%bash

MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

bwa index \
  -a bwtsw \
  $MeuDrive/referencia_tcc/hg19_tcc/hg19_tcc.fasta
```

___

**9. Fazendo a indexação com o Samtools**

```bash

%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

samtools faidx $MeuDrive/referencia_tcc/hg19_tcc/hg19_tcc.fasta

```

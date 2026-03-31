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

___

**10. Criação do Dicionário com Picard**

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

java -jar picard.jar CreateSequenceDictionary \
  REFERENCE=$MeuDrive/referencia_tcc/hg19_tcc/hg19_tcc.fasta \
  OUTPUT=$MeuDrive/referencia_tcc/hg19_tcc/hg19_tcc.dict

#Primeira linha: Falando para em java, o java ler o -jas picard.jar (se quiser, colocar só java -jar picard.jar e rodar, mantendo o bash la em cima. Ele mostra tudo que o picard pode fazer)
#Vamos usar a ferramenta/comando do picard de CreateSequenceDictionary. (se colocar java -jar picard.jar CreateSequenceDictionary, me mostra como usar)
#OUTPUT = onde eu quero salvar o meu output
#No fim, quero criar um dicionario, to passando o caminho do que quero/da minha referencia, e onde quero salvar

```

___

**11. Verificação final da preparação do genoma**

Nesta etapa, confirmamos se todos os arquivos necessários para a preparação do genoma estão presentes/foram criados da forma correta.

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

echo " Verificação final da preparação do genoma:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Verificar arquivos essenciais / É um array. Uma variavel que recebe uma lista de infos
arquivos_essenciais=(
    "hg19_tcc.fasta:Genoma FASTA"
    "hg19_tcc.fasta.fai:Índice samtools"
    "hg19_tcc.dict:Dicionário Picard"
    "hg19_tcc.fasta.amb:BWA .amb"
    "hg19_tcc.fasta.ann:BWA .ann"
    "hg19_tcc.fasta.bwt:BWA .bwt"
    "hg19_tcc.fasta.pac:BWA .pac"
    "hg19_tcc.fasta.sa:BWA .sa"
)

total=0
presentes=0
#O porque desse total e presentes?

for item in "${arquivos_essenciais[@]}"; do
    arquivo=$(echo $item | cut -d: -f1)
    descricao=$(echo $item | cut -d: -f2)
#Para cada item do array, faça....

    if [ -f "$MeuDrive/referencia_tcc/hg19_tcc/$arquivo" ]; then
        tamanho=$(du -h "$MeuDrive/referencia_tcc/hg19_tcc/$arquivo" | cut -f1)
        echo " $descricao ($tamanho)"
        ((presentes++))
    else
        echo " $descricao - AUSENTE"
    fi
    ((total++))
done

echo ""
echo "📊 RESUMO: $presentes/$total arquivos presentes"

if [ $presentes -eq $total ]; then
    echo " PREPARAÇÃO COMPLETA! Genoma pronto para uso."
else
    echo " Alguns arquivos estão faltando. Revise as etapas anteriores."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

**11.1. Extração de região do genoma de referência**

A título de teste, escolher uma região do genoma e extraí-la para ver se está tudo funcionando corretamente. Isso é feito por meio da ferramenta Samtools sob o seguinte comando: `samtools faidx "CAMINHO_DO_ARQUIVO" chrCHR_DE_INTERESSE: POSICAO_DE_INTERESSE_INICIAL - POSICAO_DE_INTERESSE_FINAL` (exemplo abaixo).

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

echo " Teste: Extraindo região chr1:1000-1100"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Testar extração de região usando samtools
samtools faidx "$MeuDrive/referencia_tcc/hg19_tcc/hg19_tcc.fasta" chr1:10000-11000

echo ""
echo " Se você viu a sequência acima, a indexação funcionou perfeitamente!"
echo " Essa região tem exatamente 101 bases (1100-1000+1)"
```
___ 

# **PREPARAÇÃO DOS ARQUIVOS FASTQ**

Fastq é o formato de arquivo de texto padrão na bioinformática para armazenar dados de sequenciamento de DNA.


## Nomeando arquivos FASTQ

O nome do arquivo `cap-ngse-a-2019_S1_L001_R1_001.fastq` segue uma convenção de [nomenclatura](https://help.basespace.illumina.com/files-used-by-basespace/fastq-files#naming). Vamos quebrar o nome em partes para entender o significado de cada elemento:

1. **`cap-ngse-b-2019`**:
   - O nome da amostra fornecido na planilha de amostras. Se um nome de amostra não for fornecido, o nome do arquivo incluirá o ID da amostra, que é um campo obrigatório na planilha de amostras e deve ser exclusivo.

2. **`S1`**:
   - Este termo geralmente representa o número da amostra ou sample number (Sample 1). Em um experimento com várias amostras, cada uma é numerada sequencialmente.

3. **`L001`**:
   - Indica a lane (faixa) usada no sequenciamento. Os sequenciadores podem dividir o fluxo de células em várias lanes, e `L001` significa que os dados vieram da primeira lane.

4. **`R1`**:
   - Refere-se à leitura (read). Em sequenciamento pareado, o DNA é sequenciado em duas direções: `R1` para a leitura forward (direta) e `R2` para a leitura reverse (invertida).

5. **`001`**:
   - O último segmento é sempre 001.

6. **`fastq`**:
   - Esta é a extensão do arquivo, indicando que o formato do arquivo é FASTQ, o que é usado para armazenar sequências de nucleotídeos junto com suas qualidades de leitura.

Então, o arquivo `cap-ngse-b-2019_S1_L001_R1_001.fastq` é um arquivo FASTQ que provavelmente contém a leitura forward de uma amostra, sequenciado na primeira lane do sequenciador.

Para fins didáticos, o número de sequências do arquivo foi reduzido, permitindo que os comandos sejam executados rapidamente. O par de FASTQ contém sequências de regiões específicas que incluem uma variante relevante para o nosso cenário clínico.

___

**12. Verificando os arquivos FASTQ**

Vamos confirmar que os arquivos de sequenciamento estão disponíveis.

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
FASTQ_DIR="$MeuDrive/Dados/fastq"

echo "📁 Verificando arquivos FASTQ..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ -d "$FASTQ_DIR" ]; then
    echo "📄 Diretório '$FASTQ_DIR' encontrado."
    if ls -lh  "$FASTQ_DIR/"*.fastq.gz 2>/dev/null; then
        echo "✅ Arquivos FASTQ encontrados:"
    else
        echo "❌ Nenhum arquivo FASTQ (ou .fastq.gz) encontrado em '$FASTQ_DIR'!"
        echo "📝 Verifique se os arquivos foram copiados corretamente."
    fi
else
    echo "❌ Diretório '$FASTQ_DIR' não encontrado!"
    echo "📝 Crie a estrutura de diretórios e copie os arquivos FASTQ."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

___

## Estrutura do arquivo FASTQ

Cada read no formato FASTQ possui **4 linhas**:

```
@SEQ_ID                    # Linha 1: Identificador da sequência
GATTTGGGGTTCAAAGCAGTATCG   # Linha 2: Sequência de bases
+                          # Linha 3: Separador (opcional: repetir ID)
!''*((((***+))%%%++)(%%%  # Linha 4: Qualidades (Phred+33)
```

### 📊 Qualidade Phred
- **Q10** = 90% de acurácia (1 erro em 10 bases)
- **Q20** = 99% de acurácia (1 erro em 100 bases)
- **Q30** = 99.9% de acurácia (1 erro em 1000 bases)

___ 

**13. Análise dos arquivos FASTQ**

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

FASTQ_DIR="$MeuDrive/Dados/fastq"

R1="cap-ngse-b-2019-chr1_S1_L001_R1_001.fastq.gz"
R2="cap-ngse-b-2019-chr1_S1_L001_R2_001.fastq.gz"

for arquivo in "$R1" "$R2"; do
    caminho="$FASTQ_DIR/$arquivo"

    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📄 Analisando arquivo: $arquivo"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

    if [ -f "$caminho" ]; then
        echo "📁 Arquivo encontrado"
        echo ""
        echo "🔍 Primeiras 12 linhas (3 reads completos):"
        zcat "$caminho" | head -12

        echo ""
        echo "📊 Estatísticas do arquivo:"
        total_linhas=$(zcat "$caminho" | wc -l)
        total_reads=$((total_linhas / 4))
        echo "• Total de linhas: $(printf "%'d" $total_linhas)"
        echo "• Total de reads:  $(printf "%'d" $total_reads)"
        echo "• Tamanho do arquivo: $(du -h "$caminho" | cut -f1)"
    else
        echo "❌ Arquivo não encontrado: $arquivo"
    fi
done

echo ""
echo "✅ Análise finalizada para R1 e R2 (FASTQ.GZ)"
```
___

**14. Alinhamento das sequencias de reads (contidas nos arquivos FASTQ) com o genoma de referência**

Comando utilizado: `BWA mem`

Pega os reads de sequenciamento, **alinha-os** ao genoma de referência, **adiciona informações importantes** sobre a amostra e **salva o resultado em um arquivo** `SAM.`


```bash

%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

SAMPLE="cap-ngse-b-2019"
Biblioteca="Exoma"
Plataforma="Illumina"

arquivo_r1="$MeuDrive/Dados/fastq/cap-ngse-b-2019-chr1_S1_L001_R1_001.fastq.gz"
arquivo_r2="$MeuDrive/Dados/fastq/cap-ngse-b-2019-chr1_S1_L001_R2_001.fastq.gz"

bwa mem -K 100000000 \
    -R "@RG\tID:$SAMPLE\tSM:$SAMPLE\tLB:$Biblioteca\tPL:$Plataforma" \
    "$MeuDrive/referencia_tcc/hg19_tcc/hg19_tcc.fasta" \
    "$arquivo_r1" \
    "$arquivo_r2" > "$MeuDrive/Dados/bam/$SAMPLE.sam"
```

**Significado das linhas do comando acima:**

**`bwa mem`:** realiza o mapeamento (alinhamento) das sequências de reads (contidas nos arquivos FASTQ). Invoca o algoritmo MEM do BWA, ideal para reads mais longas e dados paired-end.

**`-K 100000000`:** Define o tamanho do buffer de processamento para 100 milhões de bases, otimizando a performance.

**`-R "@RG\tID:$SAMPLE\tSM:$SAMPLE\tLB:$Biblioteca\tPL:$Plataforma"`:** Adiciona a informação de "Read Group" ao cabeçalho do arquivo de saída. Isso é crucial para etapas posteriores, como a remoção de duplicatas e a chamada de variantes, pois identifica a amostra, biblioteca e plataforma de sequenciamento.
> **💡 Importância:** Read Groups são **obrigatórios** para GATK e outras ferramentas!

**`@RG:`** Indica que é uma linha de Read Group.

**`ID:$SAMPLE:`** Identificador único para o grupo de reads, usando o nome da amostra ($SAMPLE).

**`SM:$SAMPLE:** `Nome da amostra ($SAMPLE).

**`LB:$Biblioteca`:** Nome da biblioteca de sequenciamento ($Biblioteca).

**`PL:$Plataforma`:** Plataforma de sequenciamento (neste caso, Illumina).

**`"$MeuDrive/referencia/hg19/hg19.fasta"`:** Especifica o caminho para o arquivo FASTA do genoma de referência.

**`"$arquivo_r1" e "$arquivo_r2"`:** Especificam os caminhos para os arquivos FASTQ das reads forward (R1) e reverse (R2) do sequenciamento paired-end.

**` > "$MeuDrive/dados/bam/$SAMPLE.sam"`:** Redireciona a saída padrão do comando BWA (que é o alinhamento no formato SAM) para um arquivo com o nome da amostra e extensão .sam, dentro do diretório de saída.

___

# **PROCESSAMENTO SAM/BAM**

## Formatos de Arquivo

| Formato | Descrição | Vantagens | Uso |
|---------|-----------|-----------|-----|
| **SAM** | Sequence Alignment/Map (texto) | Legível por humanos, debug fácil | Transferência, debug |
| **BAM** | Binary Alignment/Map (binário) | Compacto, processamento rápido | Armazenamento, análise |
| **CRAM** | Compressed Reference-oriented Alignment Map | Máxima compressão | Arquivamento longo prazo |

___

## Pipeline de Processamento

***Etapas importantes:***
* **sort**: Ordena por coordenadas genômicas. Necessário para indexação e análises subsequentes.
* **index**: Cria índice para acesso rápido. Permite acesso rápido a regiões específicas.

> **⚠️ Ordem importante:** sort → index

___ 

**15. Processar o arquivo SAM/BAM**

```bash
%%bash

MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

samtools sort -O bam -o "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam" "$MeuDrive/Dados/bam/$SAMPLE.sam" # Organiza as leituras em ordem de acordo com o mapeamento (chr1, chr2...). Porque na hora de alinhar, alinha em ordem aleatória/ -O (maiusculo) = deixar em que modo (bam, sam, cram) / O -o (minusculo), para garantir que vai salvar onde queremos com o nome que queremos / "$MeuDrive/Dados/bam/$SAMPLE.sam" - para falar que o input é esse.
samtools index "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam" # Para gerar o output quanto index / gera o .bai (a ferramenta, ja automaticamente junta aonde tá o bai). O bai é auxiliar ao bam. (assim como o fai. é auxiliar ao fasta). Precisam ser criados depois.
```

**15.1. Explorando o formato SAM/BAM**

Vamos examinar a estrutura dos arquivos gerados.


***Obs:*** Os passos do item 15 são **OPCIONAIS**! Não são essenciais para a linha de comando, mas sim para o aprendizado dos tipos de arquivo e ferramentas.


Primeiro, visualizar o arquivo no **formato SAM:**

```bash
# Para visualizar o SAM

%%bash

MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

head -5 "$MeuDrive/Dados/bam/$SAMPLE.sam"
```

Segundo, visualizar o arquivo no **formato BAM:**

Lembrando que o arquivo BAM é binário, logo, não vemos no output nada legível.

```bash
#Para visualizar o BAM (ele é binário) - não vemos no output nada legível
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

head -2 "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam"
```

**15.2. Utilizando o Samtools para olhar o arquivo BAM**

* O samtools permite com que visualizemos o arquivo em diversos formatos (até no bam).
* Nesta célula abaixo, estamos **visualizando somente o header** (por escolha)
>* -H = para mostrar só o header.

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

samtools view -H "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam"
```

Já nesta, continuamos usando o Samtools para visualizar o arquivo no formato BAM, mas não iremos solicitar ver somente o header, mas sim **os 10 primeiros alinhamentos:**

Vale destacar o que cada coluna dos arquivos SAM/BAM significam:

**`1. QNAME:`** Nome do read

**`2. FLAG:`** Informações binárias (paired, mapped, etc)

**`3. RNAME:`** Cromossomo de referência

**`4. POS:`** Posição no cromossomo (1-based)

**`5. MAPQ:`** Qualidade do mapeamento

**`6. CIGAR:`** Descrição do alinhamento

**`7. RNEXT:`** Cromossomo do par (paired-end)

**`8. PNEXT:`** Posição do par

**`9. TLEN:`** Tamanho do fragmento

**`10. SEQ:`** Sequência do read

**`11. QUAL:`** Qualidades da sequência

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

echo "🔍 Conteúdo do arquivo BAM (primeiros 10 alinhamentos):"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Explicar as colunas do SAM
echo "📊 Colunas do formato SAM/BAM:"
echo "1. QNAME: Nome do read"
echo "2. FLAG: Informações binárias (paired, mapped, etc)"
echo "3. RNAME: Cromossomo de referência"
echo "4. POS: Posição no cromossomo (1-based)"
echo "5. MAPQ: Qualidade do mapeamento"
echo "6. CIGAR: Descrição do alinhamento"
echo "7. RNEXT: Cromossomo do par (paired-end)"
echo "8. PNEXT: Posição do par"
echo "9. TLEN: Tamanho do fragmento"
echo "10. SEQ: Sequência do read"
echo "11. QUAL: Qualidades da sequência"
echo ""

echo "📄 Primeiros 10 alinhamentos:"
samtools view "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam" | head -10  #Aqui, não tá so mostrando o header (não tem o -H). O Head é so pra definir quantas linhas mostrar, não contando com o HEeader, mostrando so de alinhamentos

#OBS: no resultado, o tamanho do fragmento negativo quer dizer que o read 2 leu antes do read 1. O prof usou o Perplexity (IA) para mostrar isso.

# coordenada do local que a amostra deu match com o genoma de referência, transformando as regiões em mapeamento.
```

___

**Controle de Qualidade e Cobertura**

  **Métricas Importantes**

| Métrica | Descrição | Valor Ideal |
|---------|-----------|-------------|
| **Cobertura Média** | Média de reads por posição | >20x para exomas |
| **Cobertura Mínima** | Menor cobertura encontrada | >5x |
| **% Cobertura ≥10x** | Percentual com cobertura adequada | >95% |
| **% Cobertura ≥20x** | Percentual com boa cobertura | >90% |
| **Uniformidade** | Variação da cobertura | Baixa variação |

___ 
 **16. Transformação de BAM para BED e Análise com bedtools**
  

 **Arquivo formato BED** (Browser Extensible Data) = é um formato de texto tabular usado em bioinformática para descrever regiões genômicas.

 **bedtools** = Conjunto de ferramentas de linha de comando para trabalhar com arquivos que têm coordenadas genômicas (como BED). Suite poderosa para análise de intervalos genômicos:

* _**bamtobed**_ : Converte BAM para formato BED. Mostra so as regiões mapeadas
(que deram match), não em sequencia, mas em posição (chrx:1111-1112)

* _**merge**_: Une intervalos sobrepostos.Funde os que tem sopreposição
_**sort**_: Ordena intervalos. Ordena por posição.

* _**coverage**_: Calcula cobertura entre intervalos

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

bedtools bamtobed -i "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam" > "$MeuDrive/Dados/bam/$SAMPLE.bed" #Transforma o BAM em BED
bedtools merge -i "$MeuDrive/Dados/bam/$SAMPLE.bed" > "$MeuDrive/Dados/bam/$SAMPLE.merged.bed"
bedtools sort -i "$MeuDrive/Dados/bam/$SAMPLE.merged.bed" > "$MeuDrive/Dados/bam/$SAMPLE.sorted.bed"
```

**16.1.  Outras funcionalidades do bedtools /ETAPA OPCIONAL**

- Colocar `betools -- help`

* **Intersect** → ver a sobreposição entre arquivos (ex.: variantes do VCF que caem em regiões do BED).
* **Subtract** → tirar regiões de um arquivo com base em outro.
* **Complement** → pegar regiões do genoma que não estão cobertas pelo BED.
* **Shuffle/random** → gerar regiões aleatórias (para simulações).

```bash
  %%bash
 bedtools --help
```

**16.2. Explorando o formato BED.**

BED (Browser Extensible Data) é um formato simples para representar intervalos genômicos:

`chr8    1000    2000    read_name    score    strand`

Colunas Obrigatórias:
- Cromossomo: chr8, chr1, etc.
- Início: Posição inicial (0-based)
- Fim: Posição final (1-based)
- Colunas Opcionais:
- Nome: Identificador do intervalo
- Score: Pontuação (0-1000)
- Strand: Fita (+/-)

Para visualizar o bed, as 10 primeiras linhas:

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

head -10 "$MeuDrive/Dados/bam/$SAMPLE.sorted.bed"
```
___

**17. Cálculo de Cobertura**

Agora vamos calcular a cobertura média para cada região por meio da ferramenta bedtools, por meio do `bedtools coverage`.
* Passamos o arquivo bed que geramos, vamos passar o bam, e falar para ele calcular a cobertura de cada uma dessas regiões (ex: vai pegar chr8	6263967	6264370 e ver quantas vezes apareceu, e fazer a cobertura media de cada região)

```bash

  %%bash

MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

#calcula a cobertura média de cada região
bedtools coverage \
    -a "$MeuDrive/Dados/bam/$SAMPLE.sorted.bed" \
    -b "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam" \
    -mean > "$MeuDrive/Dados/bam/$SAMPLE.coverage.bed" # Aqui, estamos pedindo para colocar essas médias nesse aquivo

 ```

**17.1. Visualizando o arquivo BED com a cobertura média**
- Para vermos as 10 primeiras linhas, com a adição de mais uma coluna, que é a cobertura média de cada região (pedimos para fazer isso ali em cima, e ele criou esse outro arquivo bed, que pedimos. Aí aqui demos o path desse novo criado)

```bash
 %%bash
head n-10 /content/drive/MyDrive/PGBIOAGMAV/Dados/bam/cap-ngse-a-2019.coverage.bed
```

**17.2. Estatísticas gerais de todas as amostras**

* Para fazer estatisticas gerais da amostra em questão, em relação a cobertura (antes, eram a media de coberturas de cada leitura de uma amostra, e agora é geral, somando todas elas)

```bash

%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

echo "📊 Análise detalhada da cobertura:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

echo "🔍 Primeiras 10 regiões com cobertura:"
head -10 "$MeuDrive/Dados/bam/$SAMPLE.coverage.bed"

echo ""
echo "📈 Estatísticas gerais de cobertura:"

# Calcular estatísticas básicas
total_regioes=$(wc -l < "$MeuDrive/Dados/bam/$SAMPLE.coverage.bed")
cobertura_media=$(awk '{sum += $4; count++} END {printf "%.2f", sum/count}' "$MeuDrive/Dados/bam/$SAMPLE.coverage.bed")
cobertura_maxima=$(awk '{if($4 > max) max = $4} END {printf "%.2f", max}' "$MeuDrive/Dados/bam/$SAMPLE.coverage.bed")
cobertura_minima=$(awk 'NR==1{min=$4} {if($4 < min) min = $4} END {printf "%.2f", min}' "$MeuDrive/Dados/bam/$SAMPLE.coverage.bed")

echo "• Total de regiões: $(printf "%'d" $total_regioes)"
echo "• Cobertura média: ${cobertura_media}x"
echo "• Cobertura máxima: ${cobertura_maxima}x"
echo "• Cobertura mínima: ${cobertura_minima}x"

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

```

___

## **Filtragem por cobertura**

**18. Identificar regiões com cobertura ≥ 20x**
* Este valor é considerado um limiar adequado para análises de variantes.

```bash
  %%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

echo "🎯 Filtrando regiões com cobertura ≥ 20x..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Filtrar regiões com cobertura >= 20x
awk -F "\t" '$4 >= 20 {print $0}' "$MeuDrive/Dados/bam/$SAMPLE.coverage.bed" > "$MeuDrive/Dados/bam/$SAMPLE.coverage.20x.bed"

echo ""
echo "🔍 Primeiras 10 regiões com cobertura ≥ 20x:"
head -10 "$MeuDrive/Dados/bam/$SAMPLE.coverage.20x.bed"
```

___

**19. Checagem do BAM**

* Necessário para a próxima etapa, a chamada de variantes. Logo, verificar se o BAM existe, se os reads estão bem mapeados e ter uma noção rápida de cobertura:

```bash

%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

echo "📊 Análise prévia dos dados BAM..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ -f "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam" ]; then
    echo "📄 Arquivo BAM: $SAMPLE.sorted.bam"
    echo "📏 Tamanho: $(du -h "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam" | cut -f1)"

    echo ""
    echo "📈 Estatísticas básicas do BAM:"
    samtools flagstat "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam"

    echo ""
    echo "🎯 Região de cobertura (primeiras 5 posições):"
    samtools depth "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam" | head -5

    echo ""
    echo "📊 Cobertura média aproximada:"
    samtools depth "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam" | \
    awk '{sum+=$3; count++} END {printf "%.1fx (baseado em %d posições)\n", sum/count, count}'

else
    echo "❌ Arquivo BAM não encontrado!"
    echo "📝 Execute o notebook de mapeamento primeiro."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

___


# **CHAMADA DE VARIANTES**

**TIPOS PRINCIPAIS DE VARIANTES**

| Tipo | Descrição | Exemplo |
|------|-----------|---------|
| **SNP** | Single Nucleotide Polymorphism | A → G |
| **INDEL** | Inserção ou Deleção pequena | Ins: ATG, Del: -C |
| **CNV** | Copy Number Variation | Duplicação/Deleção |
| **SV** | Structural Variant | Inversão, Translocação |


Faremos isso principalmente por meio do **GATK HaplotypeCaller.**

Ele é um dos programas mais usados para chamar variantes (SNPs e indels) a partir de um BAM/CRAM alinhado ao genoma de referência.

* ✅ Abordagem de haplótipos (mais precisa que métodos baseados em posição)

* ✅ Detecção simultânea de SNPs e INDELs

* ✅ Algoritmo de reassembly local

* ✅ Compatibilidade com GATK Best Practices

📊 Etapas do Algoritmo:

* Detecção de Regiões Ativas: Identifica áreas com evidência de variação
* Reassembly Local: Reconstrói haplótipos candidatos
* Realinhamento: Alinha reads aos haplótipos candidatos
* Genotyping: Calcula probabilidades de genótipos
* Filtragem: Aplica filtros de qualidade

## **VCF**

**VCF** é o formato padrão para representar variantes genômicas.


**Estrutura Básica:**
```
##fileformat=VCFv4.2
##INFO=<ID=DP,Number=1,Type=Integer,Description="Total Depth">
#CHROM  POS     ID      REF ALT QUAL    FILTER  INFO    FORMAT  SAMPLE
chr8    12345   .       A   G   60.0    PASS    DP=30   GT:DP   0/1:30
```

**Colunas Essenciais:**
- **CHROM**: Cromossomo
- **POS**: Posição (1-based)
- **REF**: Alelo de referência
- **ALT**: Alelo alternativo
- **QUAL**: Qualidade da chamada
- **FILTER**: Status do filtro
- **INFO**: Informações adicionais
- **FORMAT**: Formato dos dados da amostra
- **SAMPLE**: Dados específicos da amostra
___

**GATK HAPLOTYPECALLER**

O que faz? Realiza a chamada de variantes e nos dá o arquivo output **VCF**. Ele identifica regiões ativas (com evidência de variação), monta essas regiões usando um grafo de De Bruijn para determinar haplótipos e alinha as leituras (reads) para calcular a probabilidade dos genótipos.

*Por que GATK HaplotypeCaller?*

- **Abordagem de haplótipos** (mais precisa que métodos baseados em posição)
- **Detecção simultânea** de SNPs e INDELs
- **Algoritmo de reassembly** local
- **Compatibilidade** com GATK Best Practices


*Etapas do Algoritmo:*

1. **Detecção de Regiões Ativas**: Identifica áreas com evidência de variação
2. **Reassembly Local**: Reconstrói haplótipos candidatos
3. **Realinhamento**: Alinha reads aos haplótipos candidatos
4. **Genotyping**: Calcula probabilidades de genótipos
5. **Filtragem**: Aplica filtros de qualidade

Obs: Caso eu queira ver as funcionalidades desta ferramenta, só rodar:
```
!./gatk-4.1.8.1/gatk HaplotypeCaller
```

**Configuração do HaplotypeCaller**

*Parâmetros Importantes:*

| Parâmetro | Descrição | Valor Recomendado |
|-----------|-----------|-------------------|
| `-R` | Genoma de referência | hg19.fasta |
| `-I` | Arquivo BAM de entrada | sorted.bam |
| `-O` | Arquivo VCF de saída | output.vcf |
| `--emit-ref-confidence` | Modo de saída | GVCF (recomendado) |
| `--min-base-quality-score` | Qualidade mínima de base | 20 |
| `--standard-min-confidence-threshold-for-calling` | Threshold para chamada | 30.0 |

*Modos de Operação possíveis:*
- **VCF Mode**: Saída apenas com variantes chamadas
- **GVCF Mode**: Inclui informações de referência (recomendado para análises futuras)

**20. Chamada de variantes com GATK HaplotypeCaller**

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

./gatk-4.1.8.1/gatk HaplotypeCaller \
    -R "$MeuDrive/referencia_tcc/hg19_tcc/hg19_tcc.fasta" \
    -I "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam" \
    -O "$MeuDrive/Dados/vcf/$SAMPLE.vcf" \
    --min-base-quality-score 20 \
    --standard-min-confidence-threshold-for-calling 30.0
```

___

**21. Explorar o formato VCF**

Código para examinar a estrutura e conteúdo do arquivo VCF gerado.

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

echo "📄 Análise do arquivo VCF gerado..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

#examinar a estrutura e conteúdo do arquivo VCF gerado

if [ -f "$MeuDrive/Dados/vcf/$SAMPLE.vcf" ]; then
    echo "📊 Informações básicas do arquivo:"
    echo "• Localização: $MeuDrive/Dados/vcf/$SAMPLE.vcf"
    echo "• Tamanho: $(du -h "$MeuDrive/Dados/vcf/$SAMPLE.vcf" | cut -f1)"
    echo "• Total de linhas: $(wc -l < "$MeuDrive/Dados/vcf/$SAMPLE.vcf")"

    echo ""
    echo "📋 Estrutura do VCF:"
    linhas_header=$(grep -c '^#' "$MeuDrive/Dados/vcf/$SAMPLE.vcf")
    linhas_dados=$(grep -c '^[^#]' "$MeuDrive/Dados/vcf/$SAMPLE.vcf")
    echo "• Linhas de cabeçalho: $linhas_header"
    echo "• Linhas de dados: $linhas_dados"

    echo ""
    echo "🔍 Cabeçalho do VCF (primeiras 20 linhas):"
    head -20 "$MeuDrive/Dados/vcf/$SAMPLE.vcf"

else
    echo "❌ Arquivo VCF não encontrado!"
    echo "📝 Execute a célula de chamada de variantes primeiro."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

___

**22. Analisar estatísticas do VCF**

Vamos analisar quantas e que tipos (indels, SNPs...) de variantes foram identificadas, assim como a distribuição delas por qualidade (Q30).

```bash
%%bash

MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"


#analisar quantas e que tipos de variantes foram identificadas

echo "📊 Estatísticas detalhadas das variantes..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ -f "$MeuDrive/Dados/vcf/$SAMPLE.vcf" ]; then

    # Contagem geral
    echo "🔢 Contagens gerais:"
    variantes=$(grep '^[^#]' "$MeuDrive/Dados/vcf/$SAMPLE.vcf" | wc -l)
    echo "• Variantes chamadas: $(printf "%'d" $variantes)"

    if [ $variantes -gt 0 ]; then
        echo ""
        echo "🧬 Análise dos tipos de variantes:"

        # Identificar SNPs e INDELs
        snps=$(grep '^[^#]' "$MeuDrive/Dados/vcf/$SAMPLE.vcf" | \
               awk 'length($4)==1 && length($5)==1' | wc -l)
        indels=$(grep '^[^#]' "$MeuDrive/Dados/vcf/$SAMPLE.vcf" | \
                awk 'length($4)!=length($5)' | wc -l)

        echo "• SNPs (Single Nucleotide Polymorphisms): $snps"
        echo "• INDELs (Inserções/Deleções): $indels"

        # Distribuição por qualidade
        echo ""
        echo "📈 Distribuição por qualidade (QUAL):"
        echo "• QUAL ≥ 30: $(grep '^[^#]' "$MeuDrive/Dados/vcf/$SAMPLE.vcf" | awk '$6 >= 30' | wc -l)"
        echo "• QUAL ≥ 50: $(grep '^[^#]' "$MeuDrive/Dados/vcf/$SAMPLE.vcf" | awk '$6 >= 50' | wc -l)"
        echo "• QUAL ≥ 100: $(grep '^[^#]' "$MeuDrive/Dados/vcf/$SAMPLE.vcf" | awk '$6 >= 100' | wc -l)"

        echo ""
        echo "🎯 Primeiras 5 variantes identificadas:"
        grep '^[^#]' "$MeuDrive/Dados/vcf/$SAMPLE.vcf" | head -5 | \
        cut -f1-8 | column -t

    else
        echo "⚠️ Nenhuma variante identificada."
        echo "💡 Isso pode indicar:"
        echo "   • Baixa cobertura na região"
        echo "   • Parâmetros muito restritivos"
        echo "   • Região conservada no cromossomo 8"
    fi

else
    echo "❌ Arquivo VCF não encontrado!"
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

___

# FILTRAGEM E ANÁLISE DE VARIANTES

## Critérios de Filtragem

Para análises clínicas, aplicamos filtros para manter apenas variantes de alta qualidade:


### **Métricas de Qualidade Importantes:** ###

| Métrica | Descrição | Threshold Recomendado |
|---------|-----------|----------------------|
| **QUAL** | Qualidade da chamada | ≥ 30 |
| **DP** | Profundidade de cobertura | ≥ 10 |
| **GQ** | Qualidade do genótipo | ≥ 20 |
| **AD** | Allelic Depth | Balanceado para heterozigóticos |

___

## Filtragem com BCFtools

O bcftools será utilizado para *manipular e filtrar arquivos VCF*, permitindo selecionar variantes com base em **métricas de qualidade, regiões genômicas, genótipos e status de filtro**. Caso queira entender melhor as funcionalidades de tool, é só rodas uma linha de código com ```` !bcftools````.


**23. Filtrar variantes com BCFtools**

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

echo "🔍 Filtragem de variantes de alta qualidade..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ -f "$MeuDrive/Dados/vcf/$SAMPLE.vcf" ]; then

    echo "📋 Aplicando filtros de qualidade:"
    echo "• QUAL ≥ 100 (qualidade da chamada)"
    echo ""

    # Aplicar filtros de qualidade
    bcftools filter -i 'QUAL>=100' "$MeuDrive/Dados/vcf/$SAMPLE.vcf" > "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf"

    echo "✅ Filtragem concluída!"
    echo ""

    # Estatísticas antes e depois da filtragem
    echo "📊 Comparação antes/depois da filtragem:"

    variantes_total=$(bcftools view -H "$MeuDrive/Dados/vcf/$SAMPLE.vcf" | wc -l)
    variantes_filtradas=$(bcftools view -H "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf" | wc -l)

    echo "• Variantes antes da filtragem: $variantes_total"
    echo "• Variantes após filtragem: $variantes_filtradas"

    if [ $variantes_total -gt 0 ]; then
        percentual=$(awk "BEGIN {printf \"%.1f\", ($variantes_filtradas/$variantes_total)*100}")
        echo "• Percentual mantido: $percentual%"
    fi

    # Mostrar variantes filtradas se existirem
    if [ $variantes_filtradas -gt 0 ]; then
        echo ""
        echo "🎯 Variantes de alta qualidade identificadas:"
        bcftools view -H "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf" | head -10 | \
        cut -f1-8 | column -t
    else
        echo ""
        echo "⚠️ Nenhuma variante passou pelos filtros de qualidade."
        echo "💡 Sugestões:"
        echo "   • Reduzir threshold de qualidade (QUAL < 100)"
        echo "   • Verificar cobertura da região"
    fi

else
    echo "❌ Arquivo VCF não encontrado!"
    echo "📝 Execute a chamada de variantes primeiro."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

***Obs:*** Arquivos VCF podem ser armazenados como texto simples (.vcf), comprimidos (.vcf.gz) ou em formato binário (.bcf).
Ferramentas como o bcftools permitem a visualização e manipulação direta desses arquivos sem a necessidade de descompactá-los manualmente.
_______


### **Interpretação detalhada do VCF + Análise estatística com vcftools** ###

Vamos explorar a estrutura e significado das informações no arquivo VCF, e também vamos usar **vcftools** para extrair estatísticas mais detalhadas das variantes.


**24. Interpretação do VCF e Análise estatística**

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

echo "📋 Interpretação detalhada do formato VCF..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ -f "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf" ]; then

    echo "📖 Estrutura das colunas VCF:"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "1. CHROM  : Cromossomo"
    echo "2. POS    : Posição (1-based)"
    echo "3. ID     : Identificador (rs number)"
    echo "4. REF    : Alelo de referência"
    echo "5. ALT    : Alelo alternativo"
    echo "6. QUAL   : Qualidade da chamada (Phred score)"
    echo "7. FILTER : Status do filtro (PASS/FAIL)"
    echo "8. INFO   : Informações adicionais"
    echo "9. FORMAT : Formato dos dados da amostra"
    echo "10. SAMPLE: Dados específicos da amostra"
    echo ""

    # Mostrar linha de header das colunas
    echo "📄 Cabeçalho das colunas:"
    grep '^#CHROM' "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf"
    echo ""

    # Análise de uma variante específica (se existir)
    if [ $(bcftools view -H "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf" | wc -l) -gt 0 ]; then
        echo "🔍 Análise detalhada da primeira variante:"
        variante=$(bcftools view -H "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf" | head -1)

        # Separar campos
        chrom=$(echo "$variante" | cut -f1)
        pos=$(echo "$variante" | cut -f2)
        ref=$(echo "$variante" | cut -f4)
        alt=$(echo "$variante" | cut -f5)
        qual=$(echo "$variante" | cut -f6)
        filter=$(echo "$variante" | cut -f7)

        echo "• Localização: $chrom:$pos"
        echo "• Mudança: $ref → $alt"
        echo "• Qualidade: $qual"
        echo "• Status: $filter"

        # Determinar tipo de variante
        if [ ${#ref} -eq 1 ] && [ ${#alt} -eq 1 ]; then
            echo "• Tipo: SNP (Single Nucleotide Polymorphism)"
        elif [ ${#ref} -gt ${#alt} ]; then
            echo "• Tipo: Deleção ($(( ${#ref} - ${#alt} )) base(s))"
        elif [ ${#ref} -lt ${#alt} ]; then
            echo "• Tipo: Inserção ($(( ${#alt} - ${#ref} )) base(s))"
        fi

        echo ""
        echo "📊 Linha completa da variante:"
        echo "$variante"

    else
        echo "ℹ️ Nenhuma variante disponível para análise detalhada."
    fi

else
    echo "❌ Arquivo VCF filtrado não encontrado!"
    echo "📝 Execute a filtragem primeiro."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

echo "📈 Análise estatística detalhada com vcftools..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ -f "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf" ]; then

    echo "🔍 Executando análises estatísticas..."

    # Estatísticas gerais
    vcftools --vcf "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf" --TsTv-summary --out "$MeuDrive/Dados/vcf/$SAMPLE"
    cat  "$MeuDrive/Dados/vcf/$SAMPLE.TsTv.summary"

else
    echo "❌ Arquivo VCF filtrado não encontrado!"
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

___

### **Visualização e validação pelo IGV** ###

Para validar as variantes chamadas, podemos visualizá-las no **IGV** junto com os dados de alinhamento.

***Arquivos Necessários para IGV:***

1. **Genoma de referência:** hg19
2. **Arquivo BAM:** `cap-ngse-b-2019.sorted.bam` + `.bai`
3. **Arquivo VCF:** `cap-ngse-b-2019.filtered.vcf`

***Instruções para IGV:***
1. **Carregar genoma:** Human hg19
2. **Carregar BAM:** Para ver alinhamentos
3. **Carregar VCF:** Para ver variantes
4. **Navegar para variantes:** Usar coordenadas das variantes

**25. Conferir arquivos disponíveis + Recomendar regiões de interesse de visualização + Passos no IGV**

Conferir se todos os arquivos necessários para o IGV estão disponíveis, selecionar regiões que podem ser interessantes de se observar pelo IGV, e elaborar um passo-a-passo para abrir os arquivos no programa.

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"

echo "👁️ Preparação para visualização no IGV..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Verificar arquivos necessários para IGV
echo "✅ Checklist de arquivos para IGV:"

arquivos_igv=(
    "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam:Arquivo BAM"
    "$MeuDrive/Dados/bam/$SAMPLE.sorted.bam.bai:Índice BAM"
    "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf:Arquivo VCF"
)

todos_presentes=true
for item in "${arquivos_igv[@]}"; do
    arquivo=$(echo $item | cut -d: -f1)
    descricao=$(echo $item | cut -d: -f2)

    if [ -f "$arquivo" ]; then
        echo "✅ $descricao"
    else
        echo "❌ $descricao - AUSENTE"
        todos_presentes=false
    fi
done

echo ""
if [ "$todos_presentes" = true ]; then
    echo "🎉 Todos os arquivos estão disponíveis para IGV!"

    # Sugerir regiões para visualização
    if [ -f "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf" ]; then
        variantes_count=$(bcftools view -H "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf" | wc -l)

        if [ $variantes_count -gt 0 ]; then
            echo ""
            echo "📍 Regiões recomendadas para visualização:"

            bcftools view -H "$MeuDrive/Dados/vcf/$SAMPLE.filtered.vcf" | head -5 | \
            while read linha; do
                chrom=$(echo "$linha" | cut -f1)
                pos=$(echo "$linha" | cut -f2)
                ref=$(echo "$linha" | cut -f4)
                alt=$(echo "$linha" | cut -f5)
                qual=$(echo "$linha" | cut -f6)

                # Criar região expandida para visualização
                start=$((pos - 50))
                end=$((pos + 50))

                echo "• $chrom:$start-$end ($ref→$alt, QUAL=$qual)"
            done
        fi
    fi

    echo ""
    echo "🔧 Passos no IGV:"
    echo "1. Genomes → Load Genome from Server → Human hg19"
    echo "2. File → Load from File → Selecionar BAM"
    echo "3. File → Load from File → Selecionar VCF"
    echo "4. Navegar para uma das regiões sugeridas acima"
    echo "5. Zoom in para ver reads individuais"

else
    echo "⚠️ Alguns arquivos estão faltando para visualização no IGV."
    echo "📝 Execute as etapas anteriores do pipeline primeiro."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

___ 

# ANOTAÇÃO FUNCIONAL

**Anotação funcional** é o processo de adicionar informações biológicas e clínicas às variantes identificadas, transformando coordenadas genômicas em conhecimento interpretável.

**Tipos de Informações Adicionadas:**

| Categoria | Informação | Exemplo |
|-----------|------------|--------|
| **Genômica** | Gene, transcript, exon | BRCA1, NM_007294.3, exon 11 |
| **Funcional** | Consequência da variante | missense, nonsense, splicing |
| **Proteica** | Mudança de aminoácido | p.Cys61Gly |
| **Frequência** | População geral | gnomAD: 0.001% |
| **Patogenicidade** | Predições computacionais | revel: deleterious |
| **Clínica** | Significado médico | ClinVar: pathogenic |

____

## Consequências Funcionais de Variantes

### Classificação por Impacto: ###

 🟢 **BAIXO IMPACTO**
- **Sinônimas**: Não alteram aminoácido (p.Leu123Leu)
- **Intrônicas**: Fora de regiões funcionais conhecidas
- **UTR**: 3' e 5' untranslated regions

 🟡 **IMPACTO MODERADO**
- **Missense**: Alteram aminoácido (p.Ala234Val)
- **Inframe indels**: Inserções/deleções múltiplas de 3bp

 🔴 **ALTO IMPACTO**
- **Nonsense**: Criam codon de parada (p.Arg456*)
- **Frameshift**: Indels que alteram frame de leitura
- **Splicing**: Afetam sítios canônicos de splicing
- **Start/stop lost**: Afetam códons de início/fim

___

### Bancos de Dados Essenciais ###

**Anotação Genômica**
- **RefSeq**: Sequências de referência (NCBI)
- **Ensembl**: Anotação genômica (EBI)
- **GENCODE**: Anotação abrangente (ENCODE)


**Frequências Populacionais**
- **gnomAD**: Genome Aggregation Database (>125k exomas)
- **1000 Genomes**: 1000 Genomes Project
- **ESP6500**: Exome Sequencing Project
- **ExAC**: Exome Aggregation Consortium


**Predições Funcionais**
- **SIFT**: Sorting Intolerant From Tolerant
- **PolyPhen-2**: Polymorphism Phenotyping v2
- **CADD**: Combined Annotation Dependent Depletion
- **REVEL**: Rare Exome Variant Ensemble Learner


**Bases Clínicas**
- **ClinVar**: Clinical significance (NCBI)
- **OMIM**: Online Mendelian Inheritance in Man
___

**26. Instalação do ANNOVAR**

**ANNOVAR** (ANNOtate VARiation) é uma das ferramentas mais populares para anotação funcional de variantes genéticas.

*Obs* = Já fizemos o ddownload do ANNOVAR  no início do código, para otimizar. Porém, para seguir um passo a passo explicativo, o ideal seria baixá-lo no momento de seu uso/funcionalidade, por fins didáticos e de entendimento. Mas isso, fica a critério do desenvolvedor. A versão que coloquei abaixo, é uma mais conduzida com instruções escritas e checkpoints, inclusive verificação se o ANNOVAR já está instalado.

```bash
echo "📥 Baixando e instalando ANNOVAR..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Verificar se ANNOVAR já está instalado
if [ -d "annovar" ]; then
    echo "✅ ANNOVAR já está instalado!"
    echo "📁 Localização: $(pwd)/annovar"
else
    echo "📥 Baixando ANNOVAR..."

    # Download do ANNOVAR (versão pública)
    wget -q http://www.openbioinformatics.org/annovar/download/0wgxR2rIVP/annovar.latest.tar.gz

    echo "📦 Extraindo ANNOVAR..."
    tar -xzf annovar.latest.tar.gz   #Tá em tar.gz, mas usamos o codigo ("tar")para deszipar

    echo "🧹 Limpando arquivo temporário..."
    rm annovar.latest.tar.gz

    echo "✅ ANNOVAR instalado com sucesso!"
fi

# Verificar instalação
if [ -f "annovar/annotate_variation.pl" ]; then  #.pl = perl (linguagem)
    echo ""
    echo "🔍 Verificando instalação:"
    echo "• Script principal: $(ls -la annovar/annotate_variation.pl | awk '{print $1, $5, $9}')"
    echo "• Scripts disponíveis: $(ls annovar/*.pl | wc -l) arquivos"

    echo ""
    echo "📋 Scripts principais do ANNOVAR:"
    ls annovar/*.pl | while read script; do
        nome=$(basename "$script")
        echo "  • $nome"
    done

else
    echo "❌ Erro na instalação do ANNOVAR!"
    echo "📝 Verifique a conectividade e tente novamente."
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

___ 
### Instalação de bancos de dados ###

**27.  Refgene**

*Obs*: Também já fizemos a instalação no **passo 5** deste script, porém, por mais que não faça diferença, esta é na etapa a qual ele será utilizado de fato, e o código está mais guiado/didático.

```bash
%%bash

#refGene: Anotação de genes RefSeq

db_name="refGene"

perl annovar/annotate_variation.pl -buildver hg19 -downdb \
    -webfrom annovar "$db_name" annovar/humandb
    #Fazendo o download do refgene, e pedindo pra salvar nessa pasta humandb dentro da pasta annovar
```

**28. Gnomad_exome**

*Obs*: Também já fizemos a instalação no **passo 5** deste script, porém, por mais que não faça diferença, esta é na etapa a qual ele será utilizado de fato, e o código está mais guiado/didático.

```bash
%%bash

MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

# gnomad_exome: Frequências do gnomAD exomas

echo "🗄️ Baixando bancos de dados essenciais..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

db_name="gnomad_exome"

if perl annovar/annotate_variation.pl -buildver hg19 -downdb \
  -webfrom annovar "$db_name" annovar/humandb 2>/dev/null; then
    echo "✅ $description baixado com sucesso"
else
    echo "⚠️ $description - erro no download"
fi

echo ""
echo "📊 Resumo do banco baixado:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"


if [ -f "annovar/humandb/hg19_$db_name.txt" ]; then
    tamanho=$(du -h "annovar/humandb/hg19_$db_name.txt" | cut -f1)
    echo "✅ $db_name ($tamanho)"
    echo "🎉 Banco $db_name pronto para anotação!"
else
    echo "❌ $db_name - não disponível"
    echo "⚠️ Problemas no download"
fi
```

**29. Revel**

*Obs*: Também já fizemos a instalação no **passo 5** deste script, porém, por mais que não faça diferença, esta é na etapa a qual ele será utilizado de fato, e o código está mais guiado/didático.


```bash
%%bash

MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

# revel: Predições funcionais

echo "🗄️ Baixando bancos de dados essenciais..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

db_name="revel"

if perl annovar/annotate_variation.pl -buildver hg19 -downdb \
  -webfrom annovar "$db_name" annovar/humandb/ 2>/dev/null; then
    echo "✅ $description baixado com sucesso"
else
    echo "⚠️ $description - erro no download"
fi

echo ""
echo "📊 Resumo do banco baixado:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"


if [ -f "annovar/humandb/hg19_$db_name.txt" ]; then
    tamanho=$(du -h "annovar/humandb/hg19_$db_name.txt" | cut -f1)
    echo "✅ $db_name ($tamanho)"
    echo "🎉 Banco $db_name pronto para anotação!"
else
    echo "❌ $db_name - não disponível"
    echo "⚠️ Problemas no download"
fi
```

**30.  Clinvar**

*Obs*: Também já fizemos a instalação no **passo 5** deste script, porém, por mais que não faça diferença, esta é na etapa a qual ele será utilizado de fato, e o código está mais guiado/didático.

```bash

%%bash

MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

# clinvar: Significado clínico das variantes

echo "🗄️ Baixando bancos de dados essenciais..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

db_name="clinvar_20200316"

if perl annovar/annotate_variation.pl -buildver hg19 -downdb \
  -webfrom annovar "$db_name" annovar/humandb/ 2>/dev/null; then
    echo "✅ $description baixado com sucesso"
else
    echo "⚠️ $description - erro no download"
fi

echo ""
echo "📊 Resumo do banco baixado:"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"


if [ -f "annovar/humandb/hg19_$db_name.txt" ]; then
    tamanho=$(du -h "annovar/humandb/hg19_$db_name.txt" | cut -f1)
    echo "✅ $db_name ($tamanho)"
    echo "🎉 Banco $db_name pronto para anotação!"
else
    echo "❌ $db_name - não disponível"
    echo "⚠️ Problemas no download"
fi
```
___

### Preparação do arquivo VCF ###

ANNOVAR trabalha com um formato específico chamado **ANNOVAR input format**, que é mais simples que o VCF.

**Formato ANNOVAR:**
```
Chr  Start  End    Ref  Alt  Otherinfo...
8    12345  12345  A    G    qual=60;...
```

**Conversão Automática:**

O ANNOVAR inclui script `convert2annovar.pl` para conversão automática.


**31. Conversão do VCF para formato ANNOVAR** 

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

echo "📄 Preparando arquivo VCF para anotação..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Verificar conteúdo do VCF
echo "🧬 Variantes no arquivo: $MeuDrive/Dados/vcf/cap-ngse-b-2019.filtered.vcf"

# Criar diretório de anotação
mkdir -p "$MeuDrive/Dados/annotation"

perl annovar/convert2annovar.pl -format vcf4 "$MeuDrive/Dados/vcf/cap-ngse-b-2019.filtered.vcf" \
    > "$MeuDrive/Dados/annotation/variantes.avinput"

# Verificar conversão
if [ -f "$MeuDrive/Dados/annotation/variantes.avinput" ]; then
    linhas_convertidas=$(wc -l < "$MeuDrive/Dados/annotation/variantes.avinput")
    echo "✅ Conversão concluída!"
    echo "📄 Arquivo gerado: variantes.avinput"
    echo "📊 Linhas convertidas: $linhas_convertidas"
else
    echo "❌ Erro na conversão para formato ANNOVAR!"
fi
```

___ 

### **Anotação Completa com table_annovar** ###

O script `table_annovar.pl` permite anotação completa em uma única execução, incluindo:


***Protocolos de Anotação:***
- **refGene**: Anotação genômica básica
- **gnomad_exome**: Frequências populacionais
- **revel**: Predições funcionais
- **clinvar**: Significado clínico


***Operações:***
- **g**: gene-based annotation
- **f**: filter-based annotation
___

**32. Anotação com table_annovar**

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

echo "🔬 Executando anotação completa com table_annovar..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

echo "🔄 Executando anotação (pode levar alguns minutos)..."

# Executar table_annovar
perl annovar/table_annovar.pl "$MeuDrive/Dados/annotation/variantes.avinput" \
    annovar/humandb/ \
    -buildver hg19 \
    -out "$MeuDrive/Dados/annotation/anotacao_completa" \
    -remove \
    -protocol "refGene,gnomad_exome,revel,clinvar_20200316" \
    -operation "g,f,f,f" \
    -nastring . \
    -csvout
#Se não tiver nada no banco, vai colocar um pontinho ( -nastring . \)
#Pedindo para gerar um csv c essas infos (csvout)
echo ""
echo "📊 Verificando resultados da anotação completa..."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# Verificar arquivo de saída
arquivo_saida="$MeuDrive/Dados/annotation/anotacao_completa.hg19_multianno.csv"

if [ -f "$arquivo_saida" ]; then
    linhas=$(wc -l < "$arquivo_saida")
    tamanho=$(du -h "$arquivo_saida" | cut -f1)

    echo "✅ Anotação completa concluída!"
    echo "📄 Arquivo: anotacao_completa.hg19_multianno.csv"
    echo "📊 Linhas: $linhas (incluindo cabeçalho)"
    echo "📏 Tamanho: $tamanho"

    echo ""
    echo "📋 Cabeçalho do arquivo (colunas disponíveis):"
    head -1 "$arquivo_saida" | tr ',' '\n' | nl | head -10

    total_colunas=$(head -1 "$arquivo_saida" | tr ',' '\n' | wc -l)
    if [ $total_colunas -gt 10 ]; then
        echo "    ... e mais $(( $total_colunas - 10 )) colunas"
    fi

else
    echo "❌ Erro na anotação completa!"
    echo "📝 Verifique se os bancos de dados foram baixados corretamente."
fi
```

___ 

# FILTRAGEM E PRIORIZAÇÃO DE VARIANTES

### Critérios de Filtragem Clínica ###

Para análise clínica, aplicamos filtros hierárquicos para priorizar variantes mais relevantes:

🔴 **ALTA prioridade**
- **Função**: exônicas, splicing
- **Tipo**: nonsense, frameshift, missense
- **Frequência**: rara (< 1% na população)
- **Predição**: deletéria (revel score >0.5)
- **Clínica**: pathogenic/likely pathogenic (ClinVar)
___

🟡 **MÉDIA prioridade**
- **Função**: UTR, promotor
- **Tipo**: sinônimas em sítios conservados
- **Frequência**: baixa (1-5%)
- **Predição**: possivelmente deletéria
___

🟢 **BAIXA prioridade**
- **Função**: intrônica, intergênica
- **Frequência**: comum (>5%)
- **Predição**: benigna
- **Clínica**: benign/likely benign

___

**33. Filtragem e priorização de variantes**

```bash
import pandas as pd
import os

# Definir caminhos de entrada e saída
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"
SAMPLE="cap-ngse-b-2019"
saida_dir = os.path.join(MeuDrive, "Dados/annotation")

# Path genoma de referência (cromossomo)
GENOMA="hg19"
CROMOSSOMO="chr1"

# Paths output
ANNOTATION_DIR=os.path.join(MeuDrive,"Dados/annotation")

# Input
MULTIANNO=os.path.join(ANNOTATION_DIR, "anotacao_completa." + GENOMA + "_multianno.csv")


# Ler arquivo CSV com pandas
df = pd.read_csv(MULTIANNO)

# Definir listas e condições para filtragem
funcoes_alta = ['exonic', 'splicing', 'exonic;splicing']
tipos_alta = ['frameshift deletion', 'frameshift insertion', 'nonsense', 'stopgain', 'stoploss']
tipos_media = ['missense', 'nonframeshift deletion', 'nonframeshift insertion']

# Normalizar colunas para lower case para busca
# Tratar NaNs para string vazia para evitar erros
df['Func.refGene'] = df['Func.refGene'].fillna('').str.lower()
df['ExonicFunc.refGene'] = df['ExonicFunc.refGene'].fillna('')

# Inicializar coluna prioridade
def classificar_prioridade(row):
    funcao = row['Func.refGene']
    tipo_exonico = row['ExonicFunc.refGene']
    if any(f in funcao for f in funcoes_alta):
        if any(t in tipo_exonico for t in tipos_alta):
            return 'alta'
        elif any(t in tipo_exonico for t in tipos_media):
            return 'media'
    return 'baixa'

# Aplicar função no dataframe
df['Prioridade'] = df.apply(classificar_prioridade, axis=1)

# Contar as prioridades
total_variantes = len(df)
contagem = df['Prioridade'].value_counts().to_dict()

# Listas separadas
alta_prioridade = df[df['Prioridade'] == 'alta']
media_prioridade = df[df['Prioridade'] == 'media']
baixa_prioridade = df[df['Prioridade'] == 'baixa']

# Imprimir resultados
print(f"📊 Resultados da priorização:")
print(f"• Total de variantes: {total_variantes}")
print(f"• 🔴 Alta prioridade: {contagem.get('alta',0)}")
print(f"• 🟡 Média prioridade: {contagem.get('media',0)}")
print(f"• 🟢 Baixa prioridade: {contagem.get('baixa',0)}")

# Mostrar variantes de alta prioridade (até 60)
if not alta_prioridade.empty:
    print("🔴 VARIANTES DE ALTA PRIORIDADE:")
    print("━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
    for i, row in alta_prioridade.head(60).iterrows():
        posicao = f"{row['Chr']}:{row['Start']}"
        mudanca = f"{row['Ref']}→{row['Alt']}"
        print(f"🧬 Variante {i+1}:")
        print(f"   📍 {posicao} ({mudanca})")
        print(f"   📝 Gene: {row.get('Gene.refGene', 'NA')}")
        print(f"   🔬 Tipo: {row['ExonicFunc.refGene']}")
        print()
        # Salvar variantes alta prioridade
    alta_prioridade.drop(columns=['Prioridade']).to_csv(
        os.path.join(saida_dir, 'variantes_alta_prioridade_tcc.csv'), index=False)
    print(f"💾 Variantes de alta prioridade salvas em: variantes_alta_prioridade_tcc.csv")
else:
    if not media_prioridade.empty:
        print("🟡 VARIANTES DE MÉDIA PRIORIDADE:")
        print("━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
        for i, row in media_prioridade.head(3).iterrows():
            posicao = f"{row['Chr']}:{row['Start']}"
            mudanca = f"{row['Ref']}→{row['Alt']}"
            print(f"🧬 Variante {i+1}:")
            print(f"   📍 {posicao} ({mudanca})")
            print(f"   📝 Gene: {row.get('Gene.refGene', 'NA')}")
            print(f"   🔬 Tipo: {row['ExonicFunc.refGene']}")
            print()
    else:
        print("ℹ️ Nenhuma variante de alta ou média prioridade identificada.")
        print("💡 Isso pode indicar:")
        print("   • Região analisada é conservada")
        print("   • Variantes são benignas ou comuns")
        print("   • Filtros podem ser muito restritivos")
```

___ 

# RELATÓRIO DE VARIANTES CLÍNICAS

Criando um relatório resumido das variantes mais relevantes para análise clínica.

___

**34. Geração de um relatório clínico**

```bash
%%bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV"

echo "📊 Geração de Relatório Clínico"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

arquivo_anotado="$MeuDrive/Dados/annotation/anotacao_completa.hg19_multianno.csv"

if [ -f "$arquivo_anotado" ]; then
    echo "📋 Gerando relatório clínico das variantes..."
    echo ""

    python3 << 'EOF'
import csv
from datetime import datetime
import os

arquivo = "/content/drive/MyDrive/PGBIOAGMAV/Dados/annotation/anotacao_completa.hg19_multianno.csv"
saida_dir = "/content/drive/MyDrive/PGBIOAGMAV/Dados/annotation"

# Criar relatório
relatorio_linhas = []
relatorio_linhas.append("# RELATÓRIO DE ANÁLISE DE VARIANTES GERMINATIVAS")
relatorio_linhas.append("")
relatorio_linhas.append(f"**Data:** {datetime.now().strftime('%d/%m/%Y %H:%M')}")
relatorio_linhas.append(f"**Amostra:** cap-ngse-b-2019")
relatorio_linhas.append(f"**Pipeline:** ANNOVAR hg19")
relatorio_linhas.append("")
relatorio_linhas.append("## RESUMO EXECUTIVO")
relatorio_linhas.append("")

# Contadores
total_variantes = 0
exonicas = 0
missense = 0
nonsense = 0
frameshift = 0
sinonimas = 0

variantes_prioritarias = []

try:
    with open(arquivo, 'r') as f:
        reader = csv.DictReader(f)

        for row in reader:
            total_variantes += 1

            funcao = row.get('Func.refGene', '').lower()
            tipo_exonico = row.get('ExonicFunc.refGene', '').lower()
            gene = row.get('Gene.refGene', 'NA')

            # Contabilizar tipos
            if 'exonic' in funcao:
                exonicas += 1

                if 'missense' in tipo_exonico:
                    missense += 1
                elif 'nonsense' in tipo_exonico or 'stopgain' in tipo_exonico:
                    nonsense += 1
                elif 'frameshift' in tipo_exonico:
                    frameshift += 1
                elif 'synonymous' in tipo_exonico:
                    sinonimas += 1

                # Variantes prioritárias (alta prioridade clínica)
                if any(x in tipo_exonico for x in ['nonsense', 'frameshift', 'stopgain', 'stoploss']):
                    variantes_prioritarias.append({
                        'gene': gene,
                        'posicao': f"{row.get('Chr', 'NA')}:{row.get('Start', 'NA')}",
                        'mudanca': f"{row.get('Ref', 'NA')}→{row.get('Alt', 'NA')}",
                        'tipo': tipo_exonico,
                        'aa_change': row.get('AAChange.refGene', 'NA')
                    })

    # Adicionar estatísticas ao relatório
    relatorio_linhas.append(f"- **Total de variantes:** {total_variantes}")
    relatorio_linhas.append(f"- **Variantes exônicas:** {exonicas}")
    relatorio_linhas.append(f"- **Variantes de alta prioridade:** {len(variantes_prioritarias)}")
    relatorio_linhas.append("")

    relatorio_linhas.append("### Distribuição por Tipo Funcional")
    relatorio_linhas.append("")
    relatorio_linhas.append(f"- Missense: {missense}")
    relatorio_linhas.append(f"- Nonsense: {nonsense}")
    relatorio_linhas.append(f"- Frameshift: {frameshift}")
    relatorio_linhas.append(f"- Sinônimas: {sinonimas}")
    relatorio_linhas.append("")

    # Variantes prioritárias
    if variantes_prioritarias:
        relatorio_linhas.append("## VARIANTES DE ALTA PRIORIDADE CLÍNICA")
        relatorio_linhas.append("")
        relatorio_linhas.append("> Variantes com alto potencial de impacto funcional")
        relatorio_linhas.append("")

        for i, var in enumerate(variantes_prioritarias[:5], 1):
            relatorio_linhas.append(f"### Variante {i}")
            relatorio_linhas.append(f"- **Gene:** {var['gene']}")
            relatorio_linhas.append(f"- **Localização:** {var['posicao']}")
            relatorio_linhas.append(f"- **Mudança:** {var['mudanca']}")
            relatorio_linhas.append(f"- **Tipo:** {var['tipo']}")
            if var['aa_change'] not in ['NA', '.', '']:
                aa_simple = var['aa_change'].split(':')[-1] if ':' in var['aa_change'] else var['aa_change']
                relatorio_linhas.append(f"- **Mudança proteica:** {aa_simple}")
            relatorio_linhas.append("")
    else:
        relatorio_linhas.append("## VARIANTES DE ALTA PRIORIDADE CLÍNICA")
        relatorio_linhas.append("")
        relatorio_linhas.append("Nenhuma variante de alta prioridade identificada nesta análise.")
        relatorio_linhas.append("")

    # Recomendações
    relatorio_linhas.append("## RECOMENDAÇÕES")
    relatorio_linhas.append("")

    if variantes_prioritarias:
        relatorio_linhas.append("1. **Validação experimental** das variantes de alta prioridade")
        relatorio_linhas.append("2. **Consulta a bases clínicas** (ClinVar, HGMD) para interpretação")
        relatorio_linhas.append("3. **Segregação familiar** se contexto clínico apropriado")
        relatorio_linhas.append("4. **Aconselhamento genético** recomendado")
    else:
        relatorio_linhas.append("1. **Revisão dos critérios** de filtragem se indicado clinicamente")
        relatorio_linhas.append("2. **Análise de variantes estruturais** complementar")
        relatorio_linhas.append("3. **Correlação clínico-laboratorial** detalhada")

    relatorio_linhas.append("")
    relatorio_linhas.append("---")
    relatorio_linhas.append("*Relatório gerado automaticamente pelo pipeline de análise genômica*")

    # Salvar relatório
    arquivo_relatorio = os.path.join(saida_dir, "relatorio_variantes_clinicas.md")
    with open(arquivo_relatorio, 'w') as f:
        f.write('\n'.join(relatorio_linhas))

    # Exibir relatório
    print('\n'.join(relatorio_linhas))

    print(f"\n💾 Relatório salvo em: relatorio_variantes_clinicas.md")

except Exception as e:
    print(f"❌ Erro na geração do relatório: {e}")
EOF
else
    echo "❌ Arquivo de anotação não encontrado!"
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

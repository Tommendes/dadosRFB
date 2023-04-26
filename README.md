# Dados Públicos CNPJ
- Fonte oficial da Receita Federal do Brasil, [aqui](https://dados.gov.br/dados/conjuntos-dados/cadastro-nacional-da-pessoa-jurdica---cnpj).
- Layout dos arquivos, [aqui](https://www.gov.br/receitafederal/dados/cnpj-metadados.pdf).

A Receita Federal do Brasil disponibiliza bases com os dados públicos do cadastro nacional de pessoas jurídicas (CNPJ). 

De forma geral, nelas constam as mesmas informações que conseguimos ver no cartão do CNPJ, quando fazemos uma consulta individual, acrescidas de outros dados de Simples Nacional, sócios e etc. Análises muito ricas podem sair desses dados, desde econômicas, mercadológicas até investigações.

# Baseado no belíssimo trabalho do gênio [Aphonso Henrique](https://github.com/aphonsoar), este repositório adiciona algumas melhorias como 
- Quebra de todos os arquivos muito grandes - não só os arquivos do SIMPLES - em partes menores durante a leitura evitando as quebras de processamento por falta de  memória durante a operação
- Adicionada uma pasta "SAVED_FILES" para armazenar dinamicamente os arquivos já importados para o BD
- Descompactar novamente os arquivos se tornou opcional

Nesse repositório consta um processo de ETL para **i)** baixar os arquivos; **ii)** descompactar; **iii)** ler e tratar, **iv)** inserir num banco de dados relacional MariaDB e **iv)** Guardar o arquivo tratado em uma pasta.

---------------------

### Infraestrutura necessária:
- [Python3](https://www.python.org/downloads/release/python-3810/)
- [MariaDB](https://mariadb.com/products/community-server/)
  
---------------------

### How to use:
1. Crie o banco de dados conforme o arquivo `banco_de_dados.sql`.

2. Defina as variáveis abaixo no corpo do arquivo code/coletar_dados_e_gravar.py conforme as variáveis de ambiente do seu ambiente de trabalho (localhost):
   - `OUTPUT_FILES_PATH`: diretório de destino para o donwload dos arquivos
   - `EXTRACTED_FILES_PATH`: diretório de destino para a extração dos arquivos .zip
   - `SAVED_FILES`: diretório de destino para os arquivos já inseridos no BD
   - `DB_USER`: usuário do banco de dados criado pelo arquivo `banco_de_dados.sql`
   - `DB_PASSWORD`: senha do usuário do BD
   - `DB_HOST`: host da conexão com o BD 
   - `DB_PORT`: porta da conexão com o BD 
   - `DB_NAME`: nome da base de dados na instância (`dados_rfb` - conforme arquivo `banco_de_dados.sql`)

3. Instale as bibliotecas necessárias, disponíveis em `requirements.txt`:
```
pip install -r requirements.txt
```

4. Execute o arquivo `coletar_dados_e_gravar.py` e aguarde a finalização do processo.
   - Os arquivos são grandes. Dependendo da infraestrutura isso deve levar muitas horas para conclusão.
   - Arquivos de 08/05/2021: `4,68 GB` compactados e `17,1 GB` descompactados.
    
---------------------

### Tabelas geradas:
- Para maiores informações, consulte o [layout](https://www.gov.br/receitafederal/pt-br/assuntos/orientacao-tributaria/cadastros/consultas/arquivos/NOVOLAYOUTDOSDADOSABERTOSDOCNPJ.pdf).
  - `empresa`: dados cadastrais da empresa em nível de matriz
  - `estabelecimento`: dados analíticos da empresa por unidade / estabelecimento (telefones, endereço, filial, etc)
  - `socios`: dados cadastrais dos sócios das empresas
  - `simples`: dados de MEI e Simples Nacional
  - `cnae`: código e descrição dos CNAEs
  - `quals`: tabela de qualificação das pessoas físicas - sócios, responsável e representante legal.  
  - `natju`: tabela de naturezas jurídicas - código e descrição.
  - `moti`: tabela de motivos da situação cadastral - código e descrição.
  - `pais`: tabela de países - código e descrição.
  - `munic`: tabela de municípios - código e descrição.


- Pelo volume de dados, as tabelas  `empresa`, `estabelecimento`, `socios` e `simples` possuem índices para a coluna `cnpj_basico`, que é a principal chave de ligação entre elas.

### Modelo de Entidade Relacionamento:
![alt text](https://github.com/aphonsoar/Receita_Federal_do_Brasil_-_Dados_Publicos_CNPJ/blob/master/Dados_RFB_ERD.png)

### Créditos especiais para:
https://github.com/aphonsoar/Receita_Federal_do_Brasil_-_Dados_Publicos_CNPJ

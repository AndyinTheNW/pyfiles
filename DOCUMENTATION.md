# Documentação de código do arquivo mailtoclient.py

   Este arquivo contém toda a documentação detalhada do programa `PyFiles` que é responsável por enviar arquivos XML e PDF para o cliente comprador e para a empresa contratante do serviço.

   Em caso de qualquer dúvida com relação ao código ou a documentação em sí, entre em contato com o desenvolvedor do programa pelo e-mail `anderson.pereira@bravocorp.com.br` ou pelo telefone `(11) 94527-0673` 
   Todos os direitos desse script são reservados a Bravo Corp. 2023.

## Importação de bibliotecas

* O programa importa as seguintes bibliotecas para seu bom funcionamento: 

- `from datetime import datetime`: Para a obtenção da data e hora atual do sistema

- `import os`: Para a manipulação de arquivos e diretórios

- `import smtplib`: Para a conexão com o servidor SMTP

- `import shutil`: Também para a manipulação de arquivos e diretórios, mas com funções diferentes das da biblioteca `os`

- `import xml.etree.ElementTree as ET`: Para a manipulação de arquivos XML

- `from email.mime.text import MIMEText`: Para a criação de mensagens de texto

- `from email.mime.multipart import MIMEMultipart`: Para a criação de mensagens com múltiplas partes

- `from email.mime.application import MIMEApplication`: Para a criação de mensagens com arquivos anexados

- `from email.mime.image import MIMEImage`: Para a criação de mensagens com imagens no corpo do email

- `from time import sleep`: Para a criação de um tempo de espera entre os ciclos de varredura dos arquivos do diretório

- `import logging`: Para a criação de logs

## Configurações iniciais

 O Progama cria um arquivo de log chamado `mailtoclient.log` no diretório `logs` para registrar as ações do programa.

## Configura o servidor SMTP e dos e-mails de origem e destino:
  
O Programa primeiro se conecta ao servidor SMTP do Outlook.com `smtp host: smtp-mail.outlook.com` na porta `587` e depois se autentica com o e-mail de origem e senha. Após isso, o programa define os endereços de e-mail de origem e destino, que são predefinidos pelo cliente.

### Configura os diretórios que serão utilizados

   O programa define o diretório inicial, que é para onde serão inicialmente enviados os arquivos XML e PDF pelo cliente, e os diretórios finais, que é para onde serão enviados os arquivos XML e PDF pelo programa, sendo eles:
        Na variável `diretorio_incial`, `todos_enviados` e `enviados_cliente`:
            - `diretorio_inicial`: Diretório onde o contratante envia os arquivos XML e PDF
            - `todos_enviados`: Diretório onde o programa envia os arquivos XML e PDF que foram enviados para contratante
            - `enviados_cliente`: Diretório onde o programa envia apenas arquivos XML e PDF que foram enviados para o cliente comprador e para o contratante.

### Variáveis nulas.
   O programa define as variáveis `cfop` e `ncm` como nulas, para que elas possam posteriormente serem sobrescritas a partir da verificação do arquivo XML. Elas precisam ser declaradas como nulas para que o programa não retorne um erro caso o arquivo XML não possua essas informações.

# Corpo Principal do programa

* Laço e a variável `cliente_email`: 
 O programa entra em um laço infinito e define a variável `cliente_email` como nula. O laço infinito é necessário para que o programa fique sempre em execução, e a variável `cliente_email` precisa ser definida dentro do laço pois ela precisa ser sobrescrita a cada ciclo de varredura do diretório inicial.

 * Verificação de arquivos XML no diretório inicial: 
 - `xml_files = [f for f in os.listdir( diretorio_inicial) if f.endswith('.xml')]`:
    O programa verifica se existem arquivos XML no diretório inicial, e caso existam, ele os adiciona a uma lista chamada `xml_files`. Caso não existam arquivos XML no diretório inicial, o programa reinicia o ciclo de varredura do diretório inicial após um tempo de espera de 5 segundos.

- Caso a variável `xml_files` esteja preenchida, o programa inicia um `for` chamado `arquivo` que realiza as seguintes declarações de variáveis:  

    1. Define a variável `caminho_arquivo` que recebe `os.path.join(diretorio_inicial, arquivo)`, que é o caminho completo do arquivo XML que será verificado (diretório inicial + variável arquivo do for)
    2. Define a variável `tree` que recebe `ET.parse(caminho_arquivo)`, que é o arquivo XML que será verificado.
    3. Define a variável `root` que recebe `tree.getroot()`, que é o elemento raiz do arquivo XML que será verificado. (elemento raiz do arquivo XML = é a tag principal do arquivo XML, no nosso caso a tag `<nfeProc>`).

- Lidando com o Namespace: Namespace é um identificador único que é atribuído a um elemento XML para que ele possa ser identificado de forma única.

    1. Define a variável `namespace`que recebe `{'ns': 'http://www.portalfiscal.inf.br/nfe'}`, que é o namespace do arquivo XML que será verificado.
    2. Define a variável `nfe_element` que recebe `root.find('ns:NFe', namespace)`, que é o elemento XML que será verificado. (elemento XML = é a tag que está dentro da tag principal do arquivo XML, no nosso caso a tag `<NFe>` que está dentro da tag `<nfeProc>`).

## Encontrando CFOP e NCM. 

* O programa começa uma sequência de diversas verificações através da condicional `if` para encontrar o CFOP e o NCM do arquivo XML que está sendo verificado, pelo caminho nfeProc → NFe → infNFe → det → prod → CFOP e nfeProc → NFe → infNFe → det → prod → NCM.
 O programa verifica se as tags do XML que serão verificadas existem, caso existam, ele navega até o CFOP, NCM e o email, e os adiciona cada um a uma variável, caso não exista alguma tag do caminho, ele define as variáveis `cfop` e `ncm` como nulas

 ### Renomeando o arquivo XML

 * O programa declara três variáveis que serão utilizadas para renomear o arquivo XML, que são:
    1. `arquivo_sem_sufixo` que recebe `arquivo.replace('nfe', '')`, que é o nome do arquivo XML sem o sufixo `nfe` 
    2.  `nome_pdf` que recebe `arquivo_sem_sufixo + '.xml', '.pdf'`, que é o nome do arquivo PDF que será renomeado
    3. `caminho_pdf` que recebe `os.path.join(diretorio_inicial, nome_pdf)`, que é o caminho completo do arquivo PDF que será renomeado (diretório inicial + nome do arquivo PDF).

### Verificando se o arquivo PDF existe

* O programa verifica se o arquivo PDF existe, caso exista, ele renomeia o arquivo PDF para o nome do arquivo XML, caso não exista, ele define a variável `nome_pdf` como nula e pula para o próximo arquivo XML, reiniciando a verificação.

### Enviando o e-mail para a empresa, e, se necessário, para o cliente.

* O programa verifica se a variável `cliente_email` não é nula, caso ela NÃO SEJA, ele entra em um `for` que segue o seguinte processo:
    1. Cria a mensagem de email com o método `MIMEMultipart()`
    2. Verifica se email_destino == `cliente_email`, se sim, ele adiciona o email do cliente, anexa a imagem de venda elaborada pelo marketing junto de assunto e corpo do e-mail próprios e devidamente personalizados. 
    3. Se email_destino não for igual a `cliente_email`, ele envia apenas para a empresa contratante, com seu assunto e corpo de e-mail diferentes, e sem imagem anexada.

### Movendo os arquivos XML e PDF para os diretórios finais

* O programa move os arquivos XML e PDF para os diretórios finais, sendo eles:
    1. `todos_enviados`: Diretório onde o programa envia os arquivos XML e PDF que foram enviados para contratante
    2. `enviados_cliente`: Diretório onde o programa envia apenas arquivos XML e PDF que foram enviados para o cliente comprador e para o contratante.

### Finalizando o programa

* O programa finaliza o ciclo de verificação do arquivo XML, e reinicia o ciclo de verificação do diretório inicial após um tempo de espera de 5 segundos.

### Funcionamento do Log e e-mail de erro do programa 

   * O programa cria um arquivo de log chamado `mailtoclient.log` que registra e armazena dados relativos ao programa, como o e-mail dos clientes encontrados, o nome do arquivo XML e PDF que foram enviados, e o horário em que o programa foi executado. Além disso, o programa envia um e-mail para o administrador do programa caso ocorra algum erro durante a execução do programa, com o horário em que o erro ocorreu e o tipo de erro. 

# Conclusão

O arquivo mailtoclient.py é responsável por monitorar um diretório de arquivos, ler e interpretar arquivos XML, renomear arquivos PDF correspondentes, e enviar esses arquivos via e-mail para um destinatário especificado no arquivo XML. O processo é contínuo e executa até que seja interrompido.

Todas as etapas e funções do programa são logadas em um arquivo chamado mailtoclient.log para permitir a rastreabilidade de eventos e ações. Além disso, em caso de falhas ou erros, o programa envia um e-mail de erro para o administrador, garantindo assim a resolução rápida de qualquer problema que possa surgir.

Este arquivo de programa é fundamental para o processo de envio de arquivos XML e PDF para o cliente comprador e a empresa contratante do serviço, garantindo que ambos recebam as informações necessárias em tempo hábil.

Por fim, qualquer dúvida ou problema encontrado em relação ao código ou a esta documentação, o contato do desenvolvedor está disponível para suporte.

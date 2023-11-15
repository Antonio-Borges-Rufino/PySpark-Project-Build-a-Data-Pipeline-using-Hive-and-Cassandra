# Descrição do Projeto
* Referencia: O projeto aqui implementado é do site projectpro.io onde estou estudando projetos de engenharia de dados profissionais. Créditos para a equipe.
* Neste projeto, usaremos um conjunto de dados de comércio eletrônico para simular os registros de compras do usuário, visualizações de produtos, histórico de carrinho e jornada do usuário na plataforma online para criar dois pipelines analíticos, Lote e Tempo Real. O processamento em lote envolverá ingestão de dados, arquitetura Lake House, processamento e visualização usando Amazon Kinesis, Glue, S3 e QuickSight para obter insights sobre o seguinte:
  * Visitantes únicos por dia
  * Durante um determinado tempo, os usuários adicionam produtos aos carrinhos, mas não os compram
  * Principais categorias por hora ou dia da semana (ou seja, para promover descontos com base em tendências)
  * Para saber quais marcas precisam de mais marketing
# Diagrama de Arquitetura
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/148a4423-b257-4d3d-993e-1d3aa0fe4e9a)
# Informação sobre os dados
* Os dados usados nesse projeto são o [eCommerce behavior data from multi category store](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store/) do kaggle
* Os dados possuem 9 colunas referente ao acesso em um site. Basicamente, o que os dados fazem, é disponibilizar o que uma pessoa faz em um acesso em uma loja de compras, por exemplo: "As 9 horas o ID_CONTA X acessou o produto XY que é um smarthone e colocou o produto no carrinho na seção Z"
* As colunas são detalhadas a seguir
  * event_time: Hora em que o evento aconteceu
  * event_type: Apenas um tipo dos eventos (view,cart,remove_from_cart,purchase)
  * product_id: Id do produto
  * category_id: Id da categoria do produto
  * category_code: Nome da categoria
  * brand: Sequencia do nome da marca em letra minuscula
  * price: Preço do produto
  * user_id: Id do usuário (estático)
  * user_session: Id da sessão (Pode variar para o mesmo usuario, cada sessão é um id único)
* Usaremos esses dados para simular como se fosse uma pessoa usando o site e então vamos fazer o pipeline acima descrito
# Instalar comando por CLI da AWS
* Vou seguir o projeto e vou usar o CLI da AWS
* Em vez de instalar na minha máquina, vou usar o AWS CloudShell, que é mais prático
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/8be6aca4-0ffc-4cc4-aee7-7d2affa1a34b)

# Criar o bucket S3
* Crie o bucket usando CLI como seguinte comando (substitua {bucket-name} pelo nome do bucket):
* ```
  aws s3 mb s3://{bucket-name}
  ```
* Podemos ver se o bucket foi criado acessando o recurso S3 e indo em bucket. No meu caso, consegui criar um bucket com acesso privado, como mostra a imagem abaixo (apaguei parte das informações)
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/5e26e951-e06e-4cd8-a981-e578a43a49ae)
* Agora vou adicionar uma tag para o bucket
* ```
  aws s3api put-bucket-tagging --bucket {bucket-name} --tagging 'TagSet=[{Key={key-tag},Value={value-tag}}]'
  ```
* Essa tag rastreia as modificações feitas pelo usuário que está consumindo o bucket
* Agora podemos enviar nossos dados para o bucket. Se eu tivesse optado pela escolha de usar o CLI do windowns, poderia fazer isso diretamente do meu pc, mas com o CloudShell é necessário ou enviar o arquivo para o CloudShell (máximo 1gb) ou enviar direto para o bucket no painel (Máximo de 5gb, que é o nivel gratuito)
* Portanto, vou enviar direto pelo painel mesmo, inclusive, vou diminuir a quantidade de dados que tenho atualmente para poder caber no nível de teste gratuito do bucket S3, como mostrado na imagem abaixo
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/7b9b8633-b6eb-4174-a86d-eb96870073ad)
* Agora é só enviar os dados (no meu caso, utilizei só as as primeiras 7 milhões de linha do arquivo) para o bucket

# Configurando e Implementando o AWS Kinesis Data Stream
* Vá no painel de serviços do AWS e pesquise por Kinesis, e então crie um serviço do tipo Kinesis Data Stream
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/b5181036-35e3-4716-b92e-bf69bd98a693)
* Existe pouca opção de criação, então optei por uso mínimo
* Após criar, o serviço fica disponivel como na imagem abaixo
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/ffbbef8b-64ac-4506-8dfe-fa56f2271f77)


# Criando Aplicação Python Que Simula Um Site
* Para criar essa aplicação, usaremos a SDK boto3 da AWS. Esse framework é preparado para rodar dentro do próprio serviço da amazon, podendo ser no CLI, no SHELL ou no Cloud9.
* Para o nosso projeto, vamos usar o Cloud9. Para isso, vá até o serviço de aplicações da AWS e procure por Cloud9. Crie um novo serviço e use um sistema de computação EC2 que esteja habilitado para o nível grátis. Use a minima quantidade de recursos possiveis.
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/94bb948d-ef5f-491e-b909-209d2e60e267)
* A imagem acima mostra o ambiente já criado, e onde clicar caso queira criar outro. Depois de criado, basta clicar em "em aberto" que é o link da IDE
* Com a IDE aberta, você instala a biblioteca na IDE através do bash no canto inferior usando o comando pip do python e depois testa com uso no bucket S3. A imagem abaixo mostra o processo
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/21d94e95-41c8-4035-adbf-b2a8e33ba2a8)
* No meu caso deu certo, ele executou o e printou os bucketes disponíveis
* Agora vamos acessar dados de dentro desses buckets. Isso é interessante porque vamos recuperar os dados do nosso bucket e vamos enviar em streaming para o Kinesis
* O código abaixo está mostrando a execução dos comandos de acesso ao bucket, com a adição do comando split('/n') podemos facilmente gerar uma matriz de strings e depois transformalos em um json que vamos enviar para o kinesis
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/b1582093-8be0-4493-af8e-34ad8a267929)
* Abaixo está o código modificado para poder transformar cada registro em um vetor. A função decode('utf-8') serve para transformar o tipo byte em string e a função split('\r\n') serve para poder quebrar de maneira correta cada registro
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/27aa5113-8498-403c-acdc-b551bd0de258)
* Agora vamos criar a função de inserção para o kimnesis englobando os recuperados do s3
* Primeiro vamos importar as bibliotecas necessárias, entre elas a biblioteca csv e json. A biblioteca csv vai ler os dados em forma de dicionário através do método DictReader() e a biblioteca json vai devolver os dados em json para o kinesis
* Abaixo está o código e o response, esse código é do projectpro, mas na documentação obtemos quase os mesmos códigos, entretanto, aqui eles fazem tratamento dos dados csv
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/20cf0000-14a9-473f-a240-7233b16d31a8)
* Primeiro usamos a biblioteca boto3, nela, podemos acessar os clientes s3 e kinesis. A partir disso, recuperamos os dados do S3 a partir do comando "response_nov = client_s3.get_object(Bucket={},Key={})". Bucket é o nome do bucket em que está os dados e Key é o nome do arquivo. O método get_object recupera qualquer tipo de arquivo do bucket.
* O código "response_nov = response_nov['Body'].read().decode("utf-8").split('\r\n')" realiza a leitura do arquivo, a resposta da requisção chega em formato JSON, e no campo Body vem os dados em forma de byte, para convertelos para string usamos decode('utf-8'). o método read() é usado para ler os dados em binário e split('\r\n') é usado para quebrar a string unica em vários vetores contendo as informações de uma só linha
* Depois criamos o for iterativo onde vamos percorrer as linhas e enviar as informações. Como o arquivo possuia cabeçalho usei o método da biblioteca csv no "for x in csv.DictReader(response):"
* Dentro do looping fazemos a transformação dos dados, aqui é importante entender o que a linha "json_load['txn_timestamp'] = datetime.now().isoformat()" está fazendo, ela salva o horário de envio da informação e a linha "response = client_kinesis.put_record(StreamName='{},Data=json.dumps(json_load, indent=4),PartitionKey=str(json_load['category_id']))" é quem de fato faz a postagem dos dados para o kinesis
* É importante resssaltar o uso da "PartitionKey" de 'category_id'. Isso é importante porque conseguimos melhorar a divisão dos nossos dados
* Podemos ver os dados chegando através do painel de obtenção de dados, como abaixo:
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/6759cbd6-ff9d-48de-a159-52fba19b8651)

# Próximo Passo
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/26890c1b-150f-4e35-a98d-deb3ed69269f)
* Os próximos passos estão relacionados a construção do processamento dos dados em streaming
* Para isso, vamos criar um banco de dados no AWS Glue utilizando o arquivo que usamos para simular os dados em stream
* Depois, vamos construir mais um canal de stream no kinesis para poder empurrar os dados do flink para ele
* Por fim, vamos construir um caderno flink que vai ler os dados do stream 1 e empurrar os dados para o stream 2
* O AWS Glue vai funcionar como registro de schema para o flink, basicamente, é ele quem vai nortear o SQL do Flink a partir do schema que ele puxar dos nossos dados em CSV guardados no S3

# Criação do AWS Glue
* Para criar um novo banco de dados no AWS Glue, entre no serviço, vá em banco de dados e aperte em criar um novo
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/9b4a5a7c-0722-416b-8daf-bb40927609f5)
* No nivel gratuito temos bastante espaço para poder utilizar o serviço
* No painel de criação, coloque o nome do seu banco de dados e a URI da sua pasta S3 onde você quer salvar o banco
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/08c031e4-78d3-418f-b1dc-f748755cb919)
* Agora vamos adicionar a tabela usando o crawler, isso significa que o glue vai analisar o arquivo e vai definir um schema pra ele
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/f611c55a-9114-40e0-a3b0-d54bfecde347)
* As configurações são pessoais, a única que deve ser realmente vista é a onfiguração que aponta para onde estão os dados. O que montamos nessa config é o crawler, então a missão dele é pesquisar dentro do bucket S3 em busca dos dados. Portanto, coloque apenas o caminho do bucket, não do dado em si
* Depois de construido, vá em tables e acesse a sua tabela AWS Glue
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/339f0fc7-468a-4b91-9e45-2650940f4736)
* Dentro da tabela, acesse os dados através do Athena
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/5434eeff-8ccf-4c47-b9e0-e0442718b3ac)
* Dentro do Athen, aperte em configurações e gerenciar, para indicar um local onde as consultas vão ficar salval. Vou salvar na mesma pasta do bucket S3 que estou usando em todo o projeto
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/5c1b812b-587c-4274-86d1-0b9fa6ce1ad9)
* Agora podemos executar a query padrão e ver os resultados
* ![image](https://github.com/Antonio-Borges-Rufino/Build-an-Analytical-Platform-for-eCommerce-using-AWS-Services/assets/86124443/508d4af7-1526-47d4-a406-53c4c899df1d)
* Pronto, nosso banco de dados AWS Glue foi criado


# Criação do Canal De Saída de Stream 2

# Criação e Integração de Aplicação com Apache Flink


  
  
  
   

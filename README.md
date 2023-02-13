# Configuração de Cenário 01

01. Criar o bucket do S3, liberar o acesso publico via ACL e habilitar a funcionalidade de site estático;
02. Upload dos arquivos de site estático de exemplo ou qualquer um que você tenha. O site de exemplo está no diretório static_website;
03. Adicionar a bucket policy publica ao seu bucket do S3.

~~~json
{
  "Id": "Policy1675826135130",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1675826133498",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "ARN_DO_SEU_BUCKET/*",
      "Principal": "*"
    }
  ]
}
~~~
04. Testar o site utilizando o endpoint de site estatico do seu bucket. Avance APENAS se o seu site foi exibido corretamente no browser;
05. Criar o certificado TLS utilizando o ACM - Amazon Certificate Manager;
06. Configuração de apontamento de DNS. Utilizei o Route53 nesse caso, mas poderia ser quaquer outro serviço de gerenciamento de zonas de DNS (Registro.br, CloudFlare, etc);
07. Criar a distribuição de CloudFront conforme o vídeo;

# Configuração de Cenário 02

01. Com o passo 01 realizado, altere a configuração de origem da sua distribuição do CloudFront para que ela passe um CUSTOM_HEADER para o seu bucket do S3;
02. Adicione a nova bucket policy ao bucket do site.

~~~json
{
  "Id": "Policy1675826890088",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1675826888949",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "ARN_DO_SEU_BUCKET/*",
      "Condition": {
        "StringEquals": {
          "aws:Referer": "CUSTOM_HEADER"
        }
      },
      "Principal": "*"
    }
  ]
}
~~~

# Configuração de Cenário 03

01. Remova a configuração de site estático do seu bucket;
02. Exclua a bucket policy adicionada anteriormente;
03. Habilite as ACLs de acesso do seu S3;
04. Altere a configuração da sua origem do CloudFront para utilizar a API REST do S3. Basta excluir o endereço de site estático do S3 e, na lista suspensa, escolher o bucket que contem os arquivos;
05. Configure uma OAC para acesso do CloudFront ao seu bucket do S3. Copie a bucket policy gerada pelo CloudFront e cole-a no seu bucket do S3. Um modelo dessa bucket policy está listado abaixo:

~~~json
{
  "Version": "2008-10-17",
  "Id": "PolicyForCloudFrontPrivateContent",
  "Statement": [
      {
          "Sid": "AllowCloudFrontServicePrincipal",
          "Effect": "Allow",
          "Principal": {
              "Service": "cloudfront.amazonaws.com"
          },
          "Action": "s3:GetObject",
          "Resource": "ARN_DO_SEU_BUCKET/*",
          "Condition": {
              "StringEquals": {
                  "AWS:SourceArn": "ARN_DA_SUA_DISTRIBUICAO_DE_CLOUDFRONT"
              }
          }
      }
  ]
}
~~~
06. Adicione no campo "default root object" do seu CloudFront o nome do seu arquivo raiz do site, normalmente index.html;
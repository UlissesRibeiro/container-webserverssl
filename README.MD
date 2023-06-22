<h1 align=center>Webserver SSL</h1>

## indice

- [Criando o container](#criando-o-container)
- [Pacotes instalados](#pacotes-instalados)
- [Criando vhost](#criando-vhost)
- [Testando](#testando)



<p>A ideia desse webserver é para validar um certificado do tipo SSL, de forma simples através do browser.</p>

## Criando o container

<p>O container será criado em cima de uma imagem do Ubuntu 20, onde vamos instalar o apache2. Mas por que não utilizar logo uma imagem do Apache ou Nginx? Eu já tive muita dor de cabeça para configurar, resolvi fazer minha própria imagem!
<br>
Rode no terminal : <br>

    docker run -dit --name webserver_test_ssl -h webserverssl -p 8443:443 -v /seu_path_de_certificados/:/path_que_quer_alocar --restart always ubuntu:20.04 
Sendo <b>/seu_path_de_certificados</b> onde estão seus certificados do tipo SSL, altere para o seu path. A minha estrutura é : <br>
    
    /certificado/
        |apache-files/
            |webserver/
                |conf.cert/
                    |streamerspb/
                        |certificado.crt

E por se tratar de SSL iremos utilizar o protocolo HTTPS, utilize uma porta referente ao protocolo, se possível uma que não esteja sendo utilizada no momento.
</p>

## Pacotes instalados

<p>Foram instalados na imagem : 
<ul>
<li>openssl</li>
<li>curl</li>
<li>net-tools</li>
<li>ssl-cert</li>
<li>apache2</li>
<li>vim</li>
</ul>

## Criando Vhost

<p>Segue o exemplo de vhost que será necessário para utilizarmos aqui no nosso projeto :
<br>
Lembre de alterar o path dos certificados, também alterar o domínio em ServerName.
<b>Obs.: Caso tenha vários certificados, faça um vhost para cada</b>

    <VirtualHost *:443>
    ServerName seu.dominio.com.br
    DocumentRoot /var/www/html

    # SSL Configuration
    SSLEngine On
    SSLCertificateFile /etc/apache2/ssl/conf.cert/certificado.crt
    SSLCertificateKeyFile /etc/apache2/ssl/conf.cert/certificado.key
    SSLCertificateChainFile /etc/apache2/ssl/conf.cert/cadeia_de_certificado.crt

    # Other Apache Configuration
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    </VirtualHost>

Seja organizado, crie um diretório para armazenar os vhosts, adote um padrão como : vhost-seu-site.conf .

<b>Obs.: adicione o host do container no seu /etc/hosts (estou trabalhando em Linux, caso esteja rodando o container em Windows, o path é \windows\system32\drivers\etc\hosts) , exemplo :

    127.0.0.1   seusite.com.br
</b>

Os vhosts, são adicionados no path <b>/etc/apache2/sites-available/</b>, para ativa-los, rode : <b>a2ensite *.conf </b> , restarte o apache .

## Testando
<p>Abra o browser de sua preferência, e na barra de endereço entre com : <b>https://seusite.com.br:8443</b> , lembre, você mapeou uma porta para HTTPS(8443), se tiver seguido todo o pass-a-passo, na barra de endereço terá o cadeado lindo de ai meu deus! 🔒 seusite.com.br:8443 , clique no cadeado para poder ver as informações do certificado.
</p>
<b>Dica : Se estiver usando linux, pode utilizar o comando : openssl s_client -connect seusite.com.br:8443, ele vai retornar as informações do certificado no terminal</b>
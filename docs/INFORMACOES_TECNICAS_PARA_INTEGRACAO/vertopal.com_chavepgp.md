---
title: Administração das chaves PGP para credenciais do Login Único
---

# Como criar um par de chaves PGP

O PGP (abreviação de Pretty Good Privacy, ou Muito Boa Privacidade) é um
programa de criptografia de chave pública e privada altamente seguro e
utilizado para proteção de conteúdo enviado por correio eletrônico.

Orientaremos como criar par de chaves PGP por meio do programa Gpg4win
(versão windows), porém existem outras aplicativos permitem geração.

Sigam os passos para criação:

1.  Realize download do aplicativo
    [Gpg4win](https://files.gpg4win.org/gpg4win-3.1.16.exe) ;
2.  Siga os passos até finalizar instalação;
3.  Execute o programa Kleopatra.

[]{.image}

4.  Selecione item do Menu **Arquivo / Novo Par de chaves**;

[]{.image}

5.  Selecione opção **Criar um par de chaves OpenPGP pessoal**;

[]{.image}

6.  Informe o **Nome** e o **Email**, marque a opção **Proteger a chave
    com senha** e clique no botão **Criar**. O nome e email deverão ser
    algum dos presentes no Item **Responsáveis pela Integração** no
    Plano de Integração

[]{.image}

7.  Concluída a criação clique no botão **Terminar**;

[]{.image}

8.  Selecione a chave criada, clique item do Menu **Arquivo / Exportar**
    para salvar chave pública;

[]{.image}

9.  Enviar chave pública junto com Plano de Integração.

# Como ler arquivo da credencial com chave PGP

Orientaremos como ler arquivo por meio do programa Gpg4win (versão
windows), porém existem outros aplicativos que podem ser empregados para
esta finalidade.

Sigam os passos para criação:

1.  Realize download do aplicativo
    [Gpg4win](https://files.gpg4win.org/gpg4win-3.1.16.exe) ;
2.  Siga os passos até finalizar instalação;
3.  Execute o programa Kleopatra.

[]{.image}

4.  Selecione item do Menu **Arquivo / Descriptografar/Verificar**;

[]{.image}

5.  Selecione o arquivo com a credencial criptografada, digite a senha
    da geração do par de chaves, clique no botão **OK**;

[]{.image}

6.  Selecione a pasta deseja salvar o aquivo descriptografado e clique
    no botão **Save all**;

[]{.image}

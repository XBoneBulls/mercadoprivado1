---
title: Passo-a-Passo para Integrar
---

# Autenticação

Para que a autenticação aconteça, todo o canal de comunicação deve ser
realizado com o protocolo HTTPS e, não utilizar a tecnologia WebView no
ambiente mobile, mas navegador nativo. Será feito um redirecionamento
para uma URL de autorização do Login Único e, após a autenticação ser
concluída, retornará um código de autenticação para a aplicação cliente
com intuito de adquirir um ticket de acesso para os serviços protegidos.

Para configuração do ambiente local dos desenvolvedores deve-se
considerar a criação de um domínio para o ambiente de desenvolvimento
das soluções clientes, por exemplo, \"local.minha_aplicacao.gov.br\"
(configurando nas máquinas dos desenvolvedores - hosts), evitando, desta
forma, a necessidade de configuração de URLs de redirecionamento e
logout fixadas em IPs dos desenvolvedores.

A utilização da autenticação do Login Único depende dos seguintes
passos:

## Passo 1

A chamada para autenticação deverá ocorrer pelo botão com o conteúdo
**Entrar com GOV.BR**. Para o formato do botão, seguir as orientações do
[Design
System](https://dsgov.estaleiro.serpro.gov.br/components/signin?tab=designer)
.

## Passo 2

Ao requisitar autenticação via Provedor, o mesmo verifica se o usuário
está logado. Caso o usuário não esteja logado o provedor redireciona
para a página de login.

## Passo 3

A requisição é feita através de um GET para o endereço
<https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/auth> passando as seguintes
informações:

  | **Variavél** | **Descrição** |
  | -------------|-------------- | 
  | **response_type** | Especifica para o provedor o tipo de autorização. Neste caso será **code** |
  | **client_id** | Chave de acesso, que identifica o serviço consumidor fornecido pelo Login Único para a aplicação cadastrada |
  | **scope** | Especifica os recursos que o serviço consumidor quer obter. Um ou mais escopos inseridos para a aplicação cadastrada. Exemplo: **openid+email+phone**. |
  | **redirect_uri** | URI de retorno cadastrada para a aplicação cliente no formato *URL Encode*. Este parâmetro não pode conter caracteres especiais conforme consta na especificação [auth 2.0 Redirection Endpoint](https://tools.ietf.org/html/rfc6749#section-3.1.2) |
  | **nonce** | Sequência de caracteres usado para associar uma sessão do serviço consumidor a um *Token* de ID e para atenuar os ataques de repetição. Pode ser um valor aleatório, mas que não seja de fácil dedução. Item obrigatório. |
  | **state** | Valor usado para manter o estado entre a solicitação e o retorno de chamada. |
  | **code_challenge** | Senha gerada pelo cliente para proteger o code da requisicao do Authorize. Seguir o padrão BASE64URL-ENCODE(SHA256(ASCII(Valor da Atributo do code_verifier a ser utilizado no /Token))). |
  | **code_challenge_method** | Será o método para proteger a senha enviada no parâmetro code_challenge. O padrão será \"S256\". |

Exemplo de requisição:

``` {.console}
https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/auth?response_type=code&client_id=ec4318d6-f797-4d65-b4f7-39a33bf4d544&scope=openid+email+phone+profile&redirect_uri=http%3A%2F%2Fappcliente.com.br%2Fphpcliente%2Floginecidadao.Php&nonce=3ed8657fd74c&state=358578ce6728b%code_challenge=K9LToxk012GYrMAwyspMMZZUdP5fpI81_vedD9dO4bI&code_challenge_method=S256
```

**Observações para Passo 3:**

-   Parâmetro **STATE** deve obrigatoriamente ser usado e deve ser
    validado no cliente (validado que foi previamente emitido pelo
    cliente)
-   Parâmetros **code_challenge e code_challenge_method** devem
    obrigatoriamente ser usado evitando que a resposta do \"authorize\"
    possa ser utilizada por um terceiro agente. Detalhes na [RFC
    PKCE](https://datatracker.ietf.org/doc/html/rfc7636)
-   Parâmetro **nonce** é obrigatório e deve ser formado por uma String aleatória. O mesmo **nonce**  estará presente no **ID Token**.

## Passo 4

Após a autorização, a requisição é retornada para a URL especificada no
redirect_uri especificada no passo anterior,
enviando os parâmetros:

  | **Variavél** | **Descrição** |
  | ------------ | ------------- |
  | **code**     | Código de autenticação gerado pelo provedor. Será utilizado para obtenção do Token de Resposta. Possui tempo de expiração e só pode ser utilizado uma única vez. |
  | **state**    | *State* gerado no [Passo 3](./vertopal.com_iniciarintegracao.md#passo-3) que pode ser utilizado para controle da aplicação cliente. Pode correlacionar com o *code* gerado. O cliente consegue saber se o CODE veio de um state gerado por ele. |

## Passo 5

Após autenticado, o provedor redireciona para a página de autorização. O
usuário habilitará o consumidor no sistema para os escopos solicitados.
Caso o usuário da solicitação autorize o acesso, é gerado um "ticket de
acesso", conforme demonstra na especificação [OpenID
Connect](https://openid.net/specs/openid-connect-core-1_0.html#TokenResponse)
;

## Passo 6

Para obter o *ticket de acesso*, o consumidor deve fazer uma requisição
POST para o endereço <https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/token> passando
as seguintes informações:

Parâmetros do Header para requisição Post
<https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/token>

  | **Variavél** | **Descrição** |
  | ------------ | ------------- |
  | **Content-Type** | Tipo do conteúdo da requisição que está sendo enviada. Nesse caso estamos enviando como um formulário |
  | **Authorization** | Informação codificada em *Base64*, no seguinte formato: CLIENT_ID:CLIENT_SECRET (senha de acesso do serviço consumidor)(utilizar [codificador para Base64](https://www.base64decode.org/) [site externo]{.image} para gerar codificação). A palavra Basic deve está antes da informação. |

Exemplo de *header*:

``` {.console}
Content-Type:application/x-www-form-urlencoded
Authorization: Basic                                            
ZWM0MzE4ZDYtZjc5Ny00ZDY1LWI0ZjctMzlhMzNiZjRkNTQ0OkFJSDRoaXBfTUJYcVJkWEVQSVJkWkdBX2dRdjdWRWZqYlRFT2NWMHlFQll4aE1iYUJzS0xwSzRzdUVkSU5FcS1kNzlyYWpaZ3I0SGJuVUM2WlRXV1lJOA==
```

Parâmetros do Body para requisição Post
<https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/token>

  | **Variavél** | **Descrição** |
  | -------------| ------------- | 
  | **grant_type** | Especifica para o provedor o tipo de autorização. Neste caso será **authorization_code** |
  | **code** | Código retornado pela requisição anterior (exemplo: Z85qv1) |
  | **redirect_uri** | URI de retorno cadastrada para a aplicação cliente. Este parâmetro não pode conter caracteres especiais conforme consta na especificação [auth 2.0 Redirection Endpoint](https://tools.ietf.org/html/rfc6749#section-3.1.2)
  | **code_verifier** | Senha sem criptografia enviada do parâmetro **code_challenge** presente no [Passo 3](./vertopal.com_iniciarintegracao.md#passo-3).  O codeVerifier deve ser uma string randômica, ter no mínimo 43 caracteres e no máximo 128. |

Exemplo de *query*

``` {.console}
curl -X POST -d 'grant_type=authorization_code&code=Z85qv1&redirect_uri=http%3A%2F%2Fappcliente.com.br%2Fphpcliente%2Floginecidadao.Php'&code_verifier='saoksdokaosdkasodkasodosdkaodksaodasodsakdaoskdsaok2212121$$2212ybhshsadju' https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/token    
```

O serviço retornará, em caso de sucesso, no formato JSON, as informações
conforme exemplo:

``` {.JSON}
{ 
    "access_token": "(Token de acesso a recursos protegidos do autenticador, bem como serviços do Login Único.)", 
    "id_token": "(Token de autenticação com informações básicas do usuário.)", 
    "token_type": "(O tipo do token gerado. Padrão: Bearer)", 
    "expires_in": "(Tempo de vida do token em segundos.)" ,
    "refresh_token": "(Token que atualiza os tokens de acesso, conforme eles expiram)",
    "refresh_expires_in": "(Tempo de vida do refresh token em segundos.)",
    "not-before-policy": "(data em formato timestamp. Qualquer token com data de publicação anterior a esta data é invalidado.)"
} 
```

**Observações para Passo 6:**

-   Tokens do Acesso gov.br devem ser preferencialmente armazenados no
    backend ou, na hipótese de necessidade de armazenamento no frontend,
    devem ser obrigatoriamente criptografados no backend;
-   A tela da aplicação cliente que recebe o parâmetro code deve
    obrigatoriamente realizar um redirect para outra página
-   A aplicação cliente deve ter sessão com mecanismo próprio, evitando
    múltiplas solicitações de autorização ao provedor de identidade do
    Acesso gov.br. O mecanismo próprio isolará a sessão da aplicação
    cliente de regras de negócio e segurança do Acesso gov.br (ou seja,
    o token do Acesso gov.br não deve ser utilizado), permitirá
    autonomia e controle próprios.
-   Parâmetro **code_verifier** deve obrigatoriamente ser usado evitando
    que a resposta do \"token\" possa ser utilizada por um terceiro
    agente. Detalhes na [RFC
    PKCE](https://datatracker.ietf.org/doc/html/rfc7636)

## Passo 7

De posse das informações do json anterior, a aplicação consumidora está
habilitada para consultar dados de recursos protegidos, que são as
informações e método de acesso do usuário ou serviços externos do Login
Único.

## Passo 8

Antes de utilizar as informações do JSON anterior, de forma especifica
os **ACCESS_TOKEN** e **ID_TOKEN**, para buscar informações referente ao
método de acesso e cadastro básico do usuário, há necessidade da
aplicação consumidora validar se as informações foram geradas pelos
serviços do Login Único. Esta validação ocorrerá por meio da consulta da
chave pública disponível no serviço
<https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/certs>. 

## Passo 9

A utilização das informações do **ACCESS_TOKEN** e **ID_TOKEN** ocorrerá
ao extrair do JSON codificado os seguintes parâmetros:

**JSON do ACCESS_TOKEN**

``` {.JSON}
{
    "sub": "(ID do usuário no Autenticador)",
    "azp": "Client ID da aplicação onde o usuário se autenticou",
    "scope": ["(Escopos autorizados pelo provedor de autenticação.)"],
    "amr": ["(Listagem dos fatores de autenticação do usuário. Pode ser “app” se logou por QR-CODE do aplicativo gov.br, “passwd” se o mesmo logou fornecendo a senha, “x509” se o mesmo utilizou certificado digital ou certificado em nuvem, ou “bank” para indicar utilização de conta bancária para autenticar. Esse último seguirá com número de identificação do banco, conforme código de compensação do Bacen presente ao final da explicação.)"],
    "iss": "(URL do provedor de autenticação que emitiu o token.)",
    "exp": "(Data/hora de expiração do token)",
    "iat": "(Data/hora em que o token foi emitido.)",
    "jti": "(Identificador único do token, reconhecido internamente pelo provedor de autenticação.)",
    "typ": "Bearer",
    "sid": "(identificador único utilizado na sessão)",
    "auth_time": "(indica quando a autenticação ocorreu)"
   
}
```

**Observações para ACCESS_TOKEN:**

-   Caso um novo método de autenticação seja adicionado, será listado no
    atributo *AMR*. As integrações devem contemplar futuras adições.
-   Caso conta do cidadão esteja com segundo fator de autenticação
    ativado, quando o atributo *AMR* vier com *passwd*, aparecerão os
    conteúdos *mfa* (indica presença de segundo fator) e *otp* (forma de
    segundo fator com código encaminhado pelo aplicativo gov.br).
-   Documento para verificação do Código de Compensação dos possíveis
    bancos integrados ao Login Único:[Documento verificar Código de
    Compensação dos Bancos](./TABELA_BACEN.pdf).

**JSON do ID_TOKEN**

``` {.JSON}
{
    "sub": "(ID do usuário no Autenticador)",
    "amr": ["(Listagem dos fatores de autenticação do usuário. Pode ser “app” se logou por QR-CODE do aplicativo gov.br, “passwd” se o mesmo logou fornecendo a senha, “x509” se o mesmo utilizou certificado digital ou certificado em nuvem, ou “bank” para indicar utilização de conta bancária para autenticar. Esse último seguirá com número de identificação do banco, conforme código de compensação do Bacen presente ao final da explicação.)"],
    "name": "(Nome cadastrado no Gov.br do usuário autenticado.)",
    "phone_number_verified": "(Confirma se o telefone foi validado no cadastro do Gov.br. Poderá ter o valor "true" ou "false")",
    "phone_number": "(Número de telefone cadastrado no Gov.br do usuário autenticado. Caso o atributo phone_number_verified do ID_TOKEN tiver o valor false, o atributo phone_number não virá no ID_TOKEN)",
    "email_verified": "(Confirma se o email foi validado no cadastro do Gov.br. Poderá ter o valor "true" ou "false")",
    "email": "(Endereço de e-mail cadastrado no Gov.br do usuário autenticado. Caso o atributo email_verified do ID_TOKEN tiver o valor false, o atributo email não virá no ID_TOKEN)",
    "cnpj": "(CNPJ vinculado ao usuário autenticado. Atributo será preenchido quando autenticação ocorrer por certificado digital de pessoal jurídica.)"
}
```

**Observações para ID_TOKEN:**

-   Os paramêtros email e phone_number não são obrigatórios. Ambos
    podem estar preenchidos ou não.
-   Caso um novo método de autenticação seja adicionado, será listado no
    atributo *AMR*. As integrações devem contemplar futuras adições.
-   Caso conta do cidadão esteja com segundo fator de autenticação
    ativado, quando o atributo *AMR* vier com *passwd*, aparecerão os
    conteúdos *mfa* (indica presença de segundo fator) e *otp* (forma de
    segundo fator com código encaminhado pelo aplicativo gov.br).
-   Documento para verificação do Código de Compensação dos possíveis
    bancos integrados ao Login Único:[Documento verificar Código de
    Compensação dos Bancos](./TABELA_BACEN.pdf).

## Passo 10

Para verificar quais níveis da conta do cidadão está localizada, deverá
acessar, pelo método GET, o serviço
<https://api.staging.acesso.gov.br/confiabilidades/v3/contas/>**cpf**/niveis?response-type=ids

Parâmetros para requisição GET
<https://api.staging.acesso.gov.br/confiabilidades/v3/contas/>**cpf**/niveis?response-type=ids

  **Variavél**        **Descrição**
  ------------------- -------------------------------------------------------------------------------------------------------
  **Authorization**   palavra **Bearer** e o *ACCESS_TOKEN* da requisição POST do <link_confiabilidades>
  **cpf**             CPF do cidadão (sem ponto, barra etc).

A resposta em caso de sucesso retorna sempre um **array** de objetos
JSON no seguinte formato:

``` {.JSON}
[
    {
    "id": "(Identificação para reconhecer o nível)",
    "dataAtualizacao": "(Mostra a data e hora que ocorreu atualização do nível na conta do usuário. A mascará será YYYY-MM-DD HH:MM:SS)"
    }
]
```

==Verificar quais níveis estão disponíveis, acesse== [Resultado Esperado do
Acesso ao Serviço de Confiabilidade Cadastral
(Níveis)](./vertopal.com_iniciarintegracao.md#resultado-esperado-do-acesso-ao-servico-de-confiabilidade-cadastral-niveis)

# Acesso ao serviço de Catálogo de Confiabilidades (Selos)

1-  Com usuário autenticado, deverá acessar, por meio do método GET ou
    POST, a URL <https://confiabilidades.staging.acesso.gov.br/>

Parâmetros da Query para requisição GET
<https://confiabilidades.staging.acesso.gov.br/>

  |**Variavél**     |     **Descrição**|
  |-----------------|------------------|
  |**client_id**    |    Chave de acesso, que identifica o serviço consumidor fornecido pelo Login Único para a aplicação cadastrada|
  |**niveis**       |     Recurso de segurança da informação da identidade, que permitem flexibilidade para realização do acesso.**Atributo opcional**| 
  |**categorias**|   Permitem manutenção mais facilitada da utilização dos níveis e confiabilidades (selos) do Login Único.**Atributo obrigatório**| |**confiabilidades**|  Consistem em orientar para qualificação das contas com a obtenção dos atributos autoritativos do cidadão a partir das bases oficias, por meio das quais permitirão a utilização da credencial de acesso em sistemas internos dos clientes e serviços providos diretamente ao cidadão. **Atributo obrigatório**|

2-  O resultado será o Catálogo apresentado com as configurações
    solicitadas. Após atendido as configurações, o Login Único devolverá
    o fluxo para aplicação por meio da URL de Lançador de Serviços,
    conforme [Plano de
    Integração](./PLANO-DE-INTEGRACAO-VERSAO4.doc).

**Observações sobre as variáveis do serviço de catálogo**

1.  ==Conteúdo para variável *niveis* : Será a informação do atributo id
    presente em cada nível no== [Resultado Esperado do Acesso ao Serviço
    de Confiabilidade Cadastral
    (Níveis)](https://manual-roteiro-integracao-login-unico.servicos.gov.br/pt/stable/iniciarintegracao.html#resultado-esperado-do-acesso-ao-servico-de-confiabilidade-cadastral-niveis)
2.  ==Contéudo para variável *confiabilidades*: Será a informação do
    atributo id presentes em cada confiabilidade no== [Resultado Esperado
    do Acesso ao Serviço de Confiabilidade Cadastral
    (Selos)](https://manual-roteiro-integracao-login-unico.servicos.gov.br/pt/stable/iniciarintegracao.html#resultado-esperado-do-acesso-ao-servico-de-confiabilidade-cadastral-selos)
3.  Tratamento do conteúdo para cada variável:

    - Todos são obrigatórios, deve-se separá-los por vírgula. **Exemplo
    (confiabilidades=301,801)**
    - Apenas um é obrigatório, deve-se separar por barra invertida.
    **Exemplo (confiabilidades=(301/801))**

# Acesso ao Serviço de Log Out

1.  **Implementação obrigatória** a fim de encerrar a sessão do usuário
    com o Login Único.
2.  Com usuário autenticado, deverá acessar, por meio do método GET ou
    POST, a URL: <https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/logout>. O acesso ao
    Log Out deverá ser pelo **Front End** da aplicação a ser integrada
    com Login Único.

Parâmetros da Query para requisição GET
<https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/logout>

  | **Variavél** | **Descrição** |
  |--------------| ------------- |
  | **post_logout_redirect_uri** | URL que direciona ao Login Único qual página deverá ser aberta quando o token for invalidado. A URL deverá ser previamente liberada por meio do preenchimento do campo **URL de Log Out** presente no [Plano de Integração](./PLANO-DE-INTEGRACAO-VERSAO4.doc). |
  | **id_token_hint**  | Cópia do ID Token gerado anteriormente em formato JWT |

Exemplo 1 de **execução** no front end em javascript

``` {.javascript}
var form = document.createElement("form");      
form.setAttribute("method", "post");
form.setAttribute("action", "https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/logout?post_logout_redirect_uri=https://www.minha-aplicacao.gov.br/retorno.html&id_token_hint=eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI3QXg2dGpHcDl...");
document.body.appendChild(form);  
form.submit();
```

Exemplo 2 de **execução** no front end em javascript

``` {.javascript}
window.location.href='https://mprivado.validacao.acesso.gov.br/auth/realms/govbrautentica/protocol/openid-connect/logout?post_logout_redirect_uri=https://www.minha-aplicacao.gov.br/retorno.html&id_token_hint=eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI3QXg2dGpHcDl...';   
```

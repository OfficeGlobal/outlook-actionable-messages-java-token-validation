---
page_type: sample
products:
- office-outlook
- office-365
languages:
- java
extensions:
  contentType: samples
  technologies:
  - Actionable messages
  platforms:
  - Android
  createdDate: 11/17/2016 2:41:50 PM
---
# Exemplo em Java de verificação de token de solicitação de ação

Os Serviços podem enviar mensagens acionáveis para usuários para que os mesmos completem tarefas simples em relação aos seus serviços. Quando um usuário executar uma das ações em uma mensagem, uma solicitação de ação será enviada pela Microsoft para o serviço. A solicitação da Microsoft conterá um token de portador no cabeçalho de autorização. Esse exemplo de código mostra como verificar o token para garantir que a solicitação da ação veio mesmo da Microsoft e como usar as declarações do token para validar a solicitação.

        @RequestMapping(value="/api/expense", method=RequestMethod.POST)
        ResponseEntity pública<?> postar(
            @RequestHeader(value="Authorization") Autenticação de string) {
            
            // Isso validará que o token foi emitido pela Microsoft à
            // URL de destino especificada, ou seja, o destino corresponde à audiência desejada (declaração de "aud" no token)
            // 
            // Em seu código, substitua https://api.contoso.com pela URL base do seu serviço.
            // Por exemplo, se a URL de destino do serviço for https://api.xyz.com/finance/expense?id=1234,
            // então, substitua https://api.contoso.com por https://api.xyz.com            
            Resultado ActionableMessageTokenValidationResult = validaToken(auth, "https://api.contoso.com");
            
            se (!result.getValidationSucceeded()) {
                se (result.getError() != nulo) {
                    System.err.println(result.getError().toString());
                }

                retorna novo ResponseEntity<>(nulo, HttpStatus. não autorizado);
            } 
            
            // Temos um token válido. Agora, verificaremos se o remetente e o executor da ação são quem
            // esperamos. O remetente é a identidade da entidade que inicialmente enviou o arquivo acionável 
            // Mensagem e o executor de ação é a identidade do usuário que realmente 
            // tomou a ação ("sub" declaração no token). 
            // 
            // Você deve substituir o código abaixo por sua própria lógica de validação 
            // Neste exemplo, verificamos se o e-mail foi enviado por expense@contoso.com (remetente esperado)
            // e o e-mail da pessoa que executou a ação é john@contoso.com (destinatário esperado)
            //
            // Você também deve devolver o cabeçalho CARD-ACTION-STATUS na resposta.
            // O valor do cabeçalho será exibido ao usuário.
            se (!result.getSender().equalsIgnoreCase("expense@contoso.com") ||
                !result.getActionPerformer().equalsIgnoreCase("john@contoso.com")) {
                Cabeçalhos HttpHeaders = novo HttpHeaders();
                resp.headers("CARD-ACTION-STATUS", "Remetente inválido ou o executor da ação não é permitido.");
                retorna novo ResponseEntity<>(nulo, HttpStatus.FORBIDDEN);
            }
            
            // Código de lógica de negócios adicional para processar o relatório de despesas.
            
            Cabeçalhos HttpHeaders = novo HttpHeaders();
            headers.add("CARD-ACTION-STATUS", "A despesa foi aprovada.");
            retorna novo ResponseEntity<>(nulo, HttpStatus.OK);
        }

        ActionableMessageTokenValidationResult validateToken privado(String authHeader, String targetUrl) {
            Resultado ActionableMessageTokenValidationResult = novo ActionableMessageTokenValidationResult();

            tentar {

                String[] tokens = authHeader.split(" ");
                se (tokens.length != 2) {
                    result.setError(new IllegalStateException("Invalid token."));
                    resultado de retorno;
                }

                Validador ActionableMessageTokenValidator = novo ActionableMessageTokenValidator();

                // O ValidateToken verificará o seguinte
                // 1. O token é emitido pela Microsoft e sua assinatura digital é válida.
                2 O token não expirou.
                3 A reivindicação do público-alvo corresponde à URL do domínio do serviço.
                resultado = validator.validateToken(tokens[1], targetUrl);
            }
            capturar (Exceção ex) {
                result.setError(ex);
            }

            resultado de retorno;
        }

O exemplo de código está usando a biblioteca a seguir para a validação de JWT.   

[SDK do OAuth 2.0 com extensões de conexão OpenID 5.17.1](https://mvnrepository.com/artifact/com.nimbusds/oauth2-oidc-sdk/5.17.1)   

Mais informações As mensagens acionáveis do Outlook estão disponíveis[aqui](https://dev.outlook.com/actions).

## Direitos autorais
Copyright (c) 2017 Microsoft. Todos os direitos reservados.


Este projeto adotou o [Código de Conduta do Código Aberto da Microsoft](https://opensource.microsoft.com/codeofconduct/). Para saber mais, confira [Perguntas frequentes sobre o Código de Conduta](https://opensource.microsoft.com/codeofconduct/faq/) ou contate [opencode@microsoft.com](mailto:opencode@microsoft.com) se tiver outras dúvidas ou comentários.

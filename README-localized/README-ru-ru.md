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
# Пример кода Java для проверки маркера запроса

Службы могут отправлять пользователям интерактивные сообщения для выполнения простых задач в отношении своих служб. При выполнении пользователем одного из действий в сообщении, ему будет отправлен запрос на обслуживание от Майкрософт. Запрос от Майкрософт будет содержать маркер носителя в заголовке авторизации. В этом примере кода показано, как проверить маркер, чтобы убедиться, что запрос получен от Майкрософт, и использовать утверждения в маркере для проверки запроса.

        @RequestMapping(value="/api/expense", method=RequestMethod.POST)
        public ResponseEntity<?> post(
            @RequestHeader(value="Authorization") String auth) {
            
            // Данное действие подтверждает, что маркер был выдан корпорацией Майкрософт
            // для указанного конечного URL-адреса, т. е. цель совпадает с целевой аудиторией (утверждение "aud" в маркере)
            // 
            // Замените в вашем коде https://api.contoso.com на базовый URL-адрес вашей службы.
            // Например, если целевой URL-адрес службы — https://api.xyz.com/finance/expense?id=1234,
            // замените https://api.contoso.com на https://api.xyz.com            
            ActionableMessageTokenValidationResult result = validateToken(auth, "https://api.contoso.com");
            
            if (!result.getValidationSucceeded()) {
                if (result.getError() != null) {
                    System.err.println(result.getError().toString());
                }

                return new ResponseEntity<>(null, HttpStatus.UNAUTHORIZED);
            } 
            
            // У нас действительный маркер. Теперь нужно убедиться, что отправитель и исполнитель действия являются теми,
            // кого мы ожидаем. Отправитель — это идентификация субъекта, который изначально отправил интерактивное 
            // сообщение, а исполнитель действия — это идентификация пользователя, который в действительности 
            выполнил действие (утверждение "sub" в маркере). 
            // 
            // Вам потребуется заменить указанный ниже код на собственную логику проверки 
            // В этом примере мы проверяем, что сообщение отправлено expense@contoso.com (ожидаемый отправитель)
            // и электронная почта лица, выполнившего действие, — john@contoso.com (ожидаемый получатель)
            //
            // Вам потребуется также вернуть заголовок CARD-ACTION-STATUS в ответе.
            // Значение заголовка будет отображаться для пользователя.
            if (!result.getSender().equalsIgnoreCase("expense@contoso.com") ||
                !result.getActionPerformer().equalsIgnoreCase("john@contoso.com")) {
                HttpHeaders headers = new HttpHeaders();
                headers.add("CARD-ACTION-STATUS", "Invalid sender or the action performer is not allowed.");
                return new ResponseEntity<>(null, headers, HttpStatus.FORBIDDEN);
            }
            
            // Дальнейший представленный здесь код бизнес-логики предназначен для обработки отчета о расходах.
            
            HttpHeaders headers = new HttpHeaders();
            headers.add("CARD-ACTION-STATUS", "The expense was approved.");
            return new ResponseEntity<>(null, headers, HttpStatus.OK);
        }

        private ActionableMessageTokenValidationResult validateToken(String authHeader, String targetUrl) {
            ActionableMessageTokenValidationResult result = new ActionableMessageTokenValidationResult();

            try {

                String[] tokens = authHeader.split(" ");
                if (tokens.length != 2) {
                    result.setError(new IllegalStateException("Invalid token."));
                    return result;
                }

                ActionableMessageTokenValidator validator = new ActionableMessageTokenValidator();

                // ValidateToken проверяет следующее
                // 1. Маркер выдан корпорацией Майкрософт и ее цифровая подпись действительна.
                // 2. Срок действия маркера не истек.
                // 3. Утверждение "Аудитория" совпадает с URL-адресом домена службы.
                result = validator.validateToken(tokens[1], targetUrl);
            }
            catch (Exception ex) {
                result.setError(ex);
            }

            return result;
        }

Образец кода использует следующую библиотеку для проверки JWT.   

[OAuth 2.0 SDK с расширениями OpenID Connect 5.17.1](https://mvnrepository.com/artifact/com.nimbusds/oauth2-oidc-sdk/5.17.1)   

Дополнительные сведения об интерактивных сообщениях в Outlook доступны [здесь](https://dev.outlook.com/actions).

## Авторские права
(c) Корпорация Майкрософт (Microsoft Corporation), 2017. Все права защищены.


Этот проект соответствует [Правилам поведения разработчиков открытого кода Майкрософт](https://opensource.microsoft.com/codeofconduct/). Дополнительные сведения см. в разделе [часто задаваемых вопросов о правилах поведения](https://opensource.microsoft.com/codeofconduct/faq/). Если у вас возникли вопросы или замечания, напишите нам по адресу [opencode@microsoft.com](mailto:opencode@microsoft.com).

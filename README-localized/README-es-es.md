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
# Ejemplo de comprobación de token de solicitud de acción en Java

Los servicios pueden enviar mensajes que requieren acción a los usuarios que realcen tareas sencillas contra sus servicios. Cuando un usuario realiza una de las acciones de un mensaje, Microsoft enviará una solicitud de acción al servicio. La solicitud de Microsoft contendrá un token de portador en el encabezado de la autorización. En este ejemplo de código se muestra cómo comprobar el token para garantizar que la solicitud de acción es de Microsoft, y usar las notificaciones del token para validar la solicitud.

        @RequestMapping(value="/api/expense", method=RequestMethod.POST)
        public ResponseEntity<?> post(
            @RequestHeader(value="Authorization") String auth) {
            
            // Esto validará que el token fue emitido por Microsoft para la
            // dirección URL de destino especificada, es decir, el destino coincide con el público deseado (notificación "aud" de token)
            // 
            // En su código, reemplace https://api.contoso.com por la dirección URL base del servicio.
            // Por ejemplo, si la dirección URL del servicio de destino es https://api.xyz.com/finance/expense?id=1234,
            // entonces reemplace https://api.contoso.com por https://api.xyz.com            
            ActionableMessageTokenValidationResult result = validateToken(auth, "https://api.contoso.com");
            
            if (!result.getValidationSucceeded()) {
                if (result.getError() != null) {
                    System.err.println(result.getError().toString());
                }

                return new ResponseEntity<>(null, HttpStatus.UNAUTHORIZED);
            } 
            
            // Ya tenemos un token válido. Ahora verificaremos que el remitente y el ejecutante de la acción sean quiénes
            // nosotros deseamos. El remitente es la identidad de la entidad que envió inicialmente el mensaje 
            // que requiere acción, y el ejecutante de la acción es la identidad del usuario que realmente 
            // realizó la acción (notificación "sub" de token) 
            // 
            // Debería reemplazar el código siguiente con su propia lógica de validación 
            // En este ejemplo, comprobamos que el correo electrónico es enviado por expense@contoso.com (remitente esperado)
            // y el correo electrónico de la persona que realizó la acción es john@contoso.com (destinatario esperado)
            //
            // También debería devolver el encabezado CARD-ACTION-STATUS en la respuesta.
            // El valor del encabezado será mostrado al usuario.
            if (!result.getSender().equalsIgnoreCase("expense@contoso.com") ||
                !result.getActionPerformer().equalsIgnoreCase("john@contoso.com")) {
                HttpHeaders headers = new HttpHeaders();
                headers.add("CARD-ACTION-STATUS", "Remitente inválido o no se permite el ejecutante de la acción.");
                return new ResponseEntity<>(null, headers, HttpStatus.FORBIDDEN);
            }
            
            // Código de lógica empresarial adicional para procesar el informe de gastos.
            
            HttpHeaders headers = new HttpHeaders();
            headers.add("CARD-ACTION-STATUS", "El gasto fue aprobado.");
            return new ResponseEntity<>(null, headers, HttpStatus.OK);
        }

        private ActionableMessageTokenValidationResult validateToken(String authHeader, String targetUrl) {
            ActionableMessageTokenValidationResult result = new ActionableMessageTokenValidationResult();

            Pruebe {

                String[] tokens = authHeader.split(" ");
                if (tokens.length != 2) {
                    result.setError(new IllegalStateException("Invalid token."));
                    return result;
                }

                ActionableMessageTokenValidator validator = new ActionableMessageTokenValidator();

                // ValidateToken comprobará lo siguiente
                // 1. El token fue emitido por Microsoft y su firma digital es válida.
                // 2. El token no ha caducado.
                // 3. La notificación del público coincide con la dirección URL del dominio del servicio.
                result = validator.validateToken(tokens[1], targetUrl);
            }
            catch (Exception ex) {
                result.setError(ex);
            }

            return result;
        }

El código de ejemplo usa la siguiente biblioteca para la validación de JWT.   

[OAuth 2.0 SDK con OpenID Connect Extensions 5.17.1](https://mvnrepository.com/artifact/com.nimbusds/oauth2-oidc-sdk/5.17.1)   

Puede encontrar más información sobre los mensajes accionables de Outlook [aquí](https://dev.outlook.com/actions).

## Copyright
Copyright (c) 2017 Microsoft. Todos los derechos reservados.


Este proyecto ha adoptado el [Código de conducta de código abierto de Microsoft](https://opensource.microsoft.com/codeofconduct/). Para obtener más información, vea [Preguntas frecuentes sobre el código de conducta](https://opensource.microsoft.com/codeofconduct/faq/) o póngase en contacto con [opencode@microsoft.com](mailto:opencode@microsoft.com) si tiene otras preguntas o comentarios.

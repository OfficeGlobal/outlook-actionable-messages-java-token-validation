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
# Exemple Java de vérification du jeton de demande d’action

Les services peuvent envoyer des messages actionnables aux utilisateurs pour effectuer des tâches simples par rapport à leurs services. Lorsqu’un utilisateur effectue l’une des actions dans un message, une demande d’action est envoyée au service par Microsoft. La demande de Microsoft contient un jeton du porteur dans l’en-tête d'autorisation. Cet exemple de code présente comment vérifier le jeton pour vous assurer que la demande d’action provient de Microsoft et utiliser les revendications dans le jeton pour valider la demande.

        @RequestMapping(value="/api/dépense", method=RequestMethod.POST)
        public ResponseEntity<?> post(
            @RequestHeader(value="Autorisation") String auth) {
            
            // Ceci valide l'émission du jeton par Microsoft pour le
            // l'URL cible spécifiée, autrement dit, la cible correspond à l’audience prévue (demande « aud » dans le jeton)
            // 
            // Dans votre code, remplacez https://api.contoso.com par l’URL de base de votre service.
            // Par exemple, si l’URL cible du service est https://api.xyz.com/finance/expense ?id=1234,
            // remplacez https://api.contoso.com par https://api.xyz.com            
            ActionableMessageTokenValidationResult result = validateToken(auth, "https://api.contoso.com");
            
            if (!result.getValidationSucceeded()) {
                if (result.getError() != null) {
                    System.err.println(result.getError().toString());
                }

                return new ResponseEntity<>(null, HttpStatus.UNAUTHORIZED);
            } 
            
            // Un jeton valide existe. Vous allez maintenant vérifier que l’expéditeur et l'exécutant de l’action sont ceux
            // prévus. L’expéditeur correspond à l’identité de l’entité qui a initialement envoyé le Message 
            // actionnable et l’exécutant de l’action correspond à l’identité de l’utilisateur qui a réellement 
            // réalisé l’action (« sous- » revendication dans le jeton). 
            // 
            // Vous devez remplacer le code ci-dessous par votre propre logique de validation. 
            // Dans cet exemple, vous vérifierez que le message électronique est envoyé par expense@contoso.com (expéditeur prévu)
            // et que l’adresse de courrier de la personne qui a effectué l’action est john@contoso.com (destinataire prévu)
            //
            // Vous devez également retourner l’en-tête CARD-ACTION-STATUS dans la réponse.
            // La valeur de l’en-tête s’affiche pour l’utilisateur.
            if (!result.getSender().equalsIgnoreCase("expense@contoso.com") ||
                !result.getActionPerformer().equalsIgnoreCase("john@contoso.com")) {
                HttpHeaders headers = new HttpHeaders();
                headers.add("CARD-ACTION-STATUS", "Expéditeur non valide ou l'exécutant de l'action n'est pas autorisé.");
                return new ResponseEntity<>(null, headers, HttpStatus.FORBIDDEN);
            }
            
            // Code de logique métier plus précis ici pour traiter le rapport sur les dépenses.
            
            HttpHeaders headers = new HttpHeaders();
            headers.add("CARD-ACTION-STATUS", "La dépense a été approuvée.");
            return new ResponseEntity<>(null, headers, HttpStatus.OK);
        }

        private ActionableMessageTokenValidationResult validateToken(String authHeader, String targetUrl) {
            ActionableMessageTokenValidationResult result = new ActionableMessageTokenValidationResult();

            try {

                String[] tokens = authHeader.split(" ");
                if (tokens.length != 2) {
                    result.setError(new IllegalStateException("Jeton non valide."));
                    return result;
                }

                ActionableMessageTokenValidator validator = new ActionableMessageTokenValidator();

                // ValidateToken vérifie les éléments suivants :
                // 1. Le jeton est émis par Microsoft et sa signature numérique est valide.
                // 2. Le jeton n’a pas expiré.
                // 3. La demande audience correspond à l’URL de domaine du service.
                result = validator.validateToken(tokens[1], targetUrl);
            }
            catch (Exception ex) {
                result.setError(ex);
            }

            return result;
        }

L’exemple de code utilise la bibliothèque suivante pour la validation JWT.   

[Kit de développement logiciel 2.0 OAuth avec extensions de connexion OpenID 5.17.1](https://mvnrepository.com/artifact/com.nimbusds/oauth2-oidc-sdk/5.17.1)   

D'autres informations sur les Messages actionnables d'Outlook sont disponibles [ici](https://dev.outlook.com/actions).

## Copyright
Copyright (c) 2017 Microsoft. Tous droits réservés.


Ce projet a adopté le [code de conduite Open Source de Microsoft](https://opensource.microsoft.com/codeofconduct/). Pour en savoir plus, reportez-vous à la [FAQ relative au code de conduite](https://opensource.microsoft.com/codeofconduct/faq/) ou contactez [opencode@microsoft.com](mailto:opencode@microsoft.com) pour toute question ou tout commentaire.

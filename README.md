Documentation des erreurs API - Service Management
1. Format des réponses d'erreur
Toutes les erreurs de l'API suivent le format RFC 7807 (Problem Details for HTTP APIs) avec des extensions spécifiques.

1.1 Structure JSON de base
json
{
  "type": "string",         // Type d'erreur (URI)
  "title": "string",        // Titre court de l'erreur
  "status": 400,            // Code HTTP
  "detail": "string",       // Description détaillée
  "instance": "string",     // URI de la ressource en erreur
  "code": "string|number",  // Code d'erreur métier
  "message": "string",      // Message utilisateur
  "path": "string",         // Chemin de la requête
  "fieldErrors": [          // Erreurs de validation (optionnel)
    {
      "objectName": "string",
      "field": "string",
      "message": "string"
    }
  ],
  "violations": [           // Violations de contraintes (optionnel)
    {
      "field": "string",
      "message": "string"
    }
  ]
}
2. Types d'erreurs gérés
2.1 Erreurs de validation (400 Bad Request)
json
{
  "type": "https://www.jhipster.tech/problem/constraint-violation",
  "title": "Method argument not valid",
  "status": 400,
  "code": "INVALID_BODY_FIELD",
  "message": "error.validation",
  "fieldErrors": [
    {
      "objectName": "user",
      "field": "email",
      "message": "doit être une adresse électronique valide"
    }
  ],
  "path": "/api/users"
}
2.2 Ressource non trouvée (404 Not Found)
json
{
  "type": "https://www.jhipster.tech/problem/entity-not-found",
  "title": "Not Found",
  "status": 404,
  "code": "RESOURCE_NOT_FOUND",
  "message": "La ressource demandée n'existe pas",
  "path": "/api/users/999"
}
2.3 Erreur de concurrence (409 Conflict)
json
{
  "type": "https://www.jhipster.tech/problem/concurrency-failure",
  "title": "Concurrency Failure",
  "status": 409,
  "code": "CONCURRENCY_FAILURE",
  "message": "error.concurrencyFailure",
  "path": "/api/users/1"
}
2.4 Erreur de validation métier (400/422)
json
{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "code": 1001,
  "message": "L'utilisateur doit être majeur",
  "path": "/api/users"
}
2.5 Erreur serveur (500 Internal Server Error)
json
{
  "type": "https://www.jhipster.tech/problem/problem-with-message",
  "title": "Internal Server Error",
  "status": 500,
  "code": 5000,
  "message": "Une erreur inattendue s'est produite",
  "path": "/api/users"
}
3. Codes d'erreur
3.1 Codes HTTP utilisés
Code	Signification	Utilisation typique
400	Bad Request	Requête mal formée ou invalide
401	Unauthorized	Authentification requise
403	Forbidden	Droits insuffisants
404	Not Found	Ressource introuvable
409	Conflict	Conflit de version (optimistic locking)
422	Unprocessable Entity	Validation sémantique échouée
500	Internal Server Error	Erreur serveur inattendue
3.2 Codes métier (exemples)
Code	Type d'erreur	Exemple de message
1001	Validation métier	"L'utilisateur doit être majeur"
2001	Contrainte d'intégrité	"L'email est déjà utilisé"
3001	Ressource non trouvée	"Utilisateur introuvable"
4001	Erreur d'authentification	"Identifiants invalides"
5001	Erreur technique	"Échec de connexion à la base de données"
4. Bonnes pratiques pour les clients
4.1 Gestion des erreurs
Toujours vérifier le code HTTP avant de traiter la réponse
Ne pas se fier uniquement au message texte, utiliser le code d'erreur pour la logique conditionnelle
Afficher les messages d'erreur à l'utilisateur quand ils sont destinés à être lus
4.2 Exemple de gestion côté client (JavaScript)
javascript
async function fetchUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      const error = await response.json();
      if (error.status === 404) {
        showToast('Utilisateur non trouvé');
      } else if (error.code === 'INVALID_BODY_FIELD') {
        // Traiter les erreurs de validation
        error.fieldErrors?.forEach(fieldError => {
          showFieldError(fieldError.field, fieldError.message);
        });
      } else {
        showGenericError(error.message);
      }
      return null;
    }
    return await response.json();
  } catch (error) {
    console.error('Erreur réseau:', error);
    showToast('Erreur de connexion au serveur');
    return null;
  }
}
5. Personnalisation des messages
Pour personnaliser les messages d'erreur, utilisez le fichier i18n/messages_fr.properties :

properties
error.validation=Erreur de validation
error.concurrencyFailure=Conflit de version
error.email.exists=Cet email est déjà utilisé
6. Journalisation
Toutes les erreurs sont journalisées côté serveur avec les informations suivantes :

Timestamp
ID de corrélation
Chemin de la requête
Méthode HTTP
Détails de l'erreur (sans informations sensibles)
7. Dépannage
Problèmes courants
Erreur 400 avec fieldErrors
Vérifiez que tous les champs requis sont fournis et valides.
Erreur 401
Vérifiez que le token d'authentification est valide et inclus dans le header Authorization.
Erreur 500
Vérifiez les logs serveur pour plus de détails.
Feedback submitted


genere moi un fichier read me pour ca pour les exemples prends le langage java

Je vais créer un fichier README.md bien structuré pour documenter la gestion des erreurs de l'API, avec des exemples en Java.

markdown
# Gestion des Erreurs - API Service Management

## Table des Matières
1. [Introduction](#introduction)
2. [Format des Réponses d'Erreur](#format-des-réponses-derreur)
3. [Types d'Erreurs](#types-derreurs)
4. [Exemples en Java](#exemples-en-java)
5. [Bonnes Pratiques](#bonnes-pratiques)
6. [Dépannage](#dépannage)

## Introduction

Ce document décrit le système de gestion des erreurs de l'API Service Management. Toutes les erreurs suivent le format standard [RFC 7807](https://tools.ietf.org/html/rfc7807) avec des extensions spécifiques.

## Format des Réponses d'Erreur

### Structure JSON

```json
{
  "type": "string",         // Type d'erreur (URI)
  "title": "string",        // Titre court
  "status": 400,            // Code HTTP
  "code": "string|number",  // Code d'erreur métier
  "message": "string",      // Message utilisateur
  "path": "string",         // Chemin de la requête
  "fieldErrors": [          // Erreurs de validation
    {
      "objectName": "string",
      "field": "string",
      "message": "string"
    }
  ]
}
Types d'Erreurs
1. Erreur de Validation (400)
Exemple de réponse :

json
{
  "type": "https://www.jhipster.tech/problem/constraint-violation",
  "title": "Bad Request",
  "status": 400,
  "code": "INVALID_BODY_FIELD",
  "message": "error.validation",
  "fieldErrors": [
    {
      "objectName": "userDTO",
      "field": "email",
      "message": "doit être une adresse électronique valide"
    }
  ]
}
2. Ressource Non Trouvée (404)
Exemple de réponse :

json
{
  "type": "https://www.jhipster.tech/problem/entity-not-found",
  "title": "Not Found",
  "status": 404,
  "code": "RESOURCE_NOT_FOUND",
  "message": "La ressource demandée n'existe pas"
}
3. Erreur Métier (400/422)
Exemple de réponse :

json
{
  "title": "Bad Request",
  "status": 400,
  "code": 1001,
  "message": "L'utilisateur doit être majeur"
}
Exemples en Java
1. Lancer une erreur de validation
java
@PostMapping("/users")
public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserDTO userDTO) {
    // Le @Valid déclenche automatiquement la validation
    UserDTO result = userService.save(userDTO);
    return ResponseEntity.ok(result);
}
2. Lancer une erreur métier personnalisée
java
@GetMapping("/users/{id}")
public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
    return userRepository.findById(id)
        .map(ResponseEntity::ok)
        .orElseThrow(() -> {
            ResponseCode rc = new ResponseCode();
            rc.setCode(3001);
            rc.setMessage("Utilisateur non trouvé avec l'ID: " + id);
            return new InvalidRequestException(rc);
        });
}
3. Gestion des erreurs dans un service
java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;

    public UserDTO createUser(UserDTO userDTO) {
        if (userRepository.existsByEmail(userDTO.getEmail())) {
            ResponseCode rc = new ResponseCode();
            rc.setCode(2001);
            rc.setMessage("L'email est déjà utilisé");
            throw new InvalidRequestException(rc);
        }
        
        if (userDTO.getAge() < 18) {
            ResponseCode rc = new ResponseCode();
            rc.setCode(1001);
            rc.setMessage("L'utilisateur doit être majeur");
            throw new InvalidRequestException(rc);
        }

        User user = userMapper.toEntity(userDTO);
        return userMapper.toDto(userRepository.save(user));
    }
}
4. Gestion des erreurs de contrainte d'intégrité
java
@ExceptionHandler
public ResponseEntity<Problem> handleDataIntegrityViolation(DataIntegrityViolationException ex, NativeWebRequest request) {
    String message = "Erreur d'intégrité des données";
    
    if (ex.getCause() instanceof ConstraintViolationException) {
        message = "Violation de contrainte: " + 
            ((ConstraintViolationException) ex.getCause())
                .getConstraintName();
    }
    
    Problem problem = Problem.builder()
        .withTitle("Erreur de données")
        .withStatus(Status.CONFLICT)
        .withDetail(message)
        .build();
        
    return create(ex, problem, request);
}
Bonnes Pratiques
Utilisez les bonnes exceptions :
Valid
 pour la validation des entrées
InvalidRequestException
 pour les erreurs métier
ResponseStatusException pour les cas simples
Messages d'erreur :
Soyez clairs et concis
Ne divulguez pas d'informations sensibles
Utilisez les codes d'erreur pour l'internationalisation
Journalisation :
Loggez les erreurs avec un niveau approprié
Incluez un identifiant de corrélation
Dépannage
Erreurs Courantes
400 Bad Request
Vérifiez le format des données envoyées
Consultez les fieldErrors pour les détails
401 Unauthorized
Vérifiez le token JWT
Assurez-vous qu'il n'est pas expiré
403 Forbidden
Vérifiez les rôles et permissions de l'utilisateur
404 Not Found
Vérifiez l'URL et les paramètres de la requête
500 Internal Server Error
Consultez les logs serveur
Vérifiez la connectivité aux services externes
Exemple de Logs
2023-11-22 20:30:45 ERROR 12345 --- [nio-8080-exec-1] s.d.d.a.s.w.r.errors.ExceptionTranslator : [123e4567-e89b-12d3-a456-426614174000] Erreur lors du traitement de la requête
org.springframework.dao.DataIntegrityViolationException: could not execute statement; SQL [n/a]; constraint [uk_user_email]
    at org.hibernate.exception.internal.SQLExceptionTypeDelegate.convert(SQLExceptionTypeDelegate.java:59)
    ...
Caused by: org.h2.jdbc.JdbcSQLIntegrityConstraintViolationException: Unique index or primary key violation: "UK_USER_EMAIL_INDEX_2 ON PUBLIC.USER(EMAIL) VALUES (/* 1 */ 'test@example.com')"
Support
Pour toute question ou problème, contactez l'équipe de développement à l'adresse support@example.com.

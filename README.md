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

{
  "type": "https://www.jhipster.tech/problem/entity-not-found",
  "title": "Not Found",
  "status": 404,
  "code": "RESOURCE_NOT_FOUND",
  "message": "La ressource demandée n'existe pas"
}

3. Erreur Métier (400/422)
Exemple de réponse :

{
  "title": "Bad Request",
  "status": 400,
  "code": 1001,
  "message": "L'utilisateur doit être majeur"
}

Exemples en Java
1. Lancer une erreur de validation

@PostMapping("/users")
public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserDTO userDTO) {
    // Le @Valid déclenche automatiquement la validation
    UserDTO result = userService.save(userDTO);
    return ResponseEntity.ok(result);
}

2. Lancer une erreur métier personnalisée

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

## Bonnes Pratiques

### Utilisez les bonnes exceptions
- `@Valid` pour la validation des entrées
- [InvalidRequestException](cci:2://file:///Users/khadim/Desktop/workspace_api/api-servicemanagement/src/main/java/sn/sonatel/dsi/dac/dif/api/servicemanagement/web/rest/errors/InvalidRequestException.java:5:0-27:1) pour les erreurs métier
- `ResponseStatusException` pour les cas simples

### Messages d'erreur
- Soyez clairs et concis
- Ne divulguez pas d'informations sensibles
- Utilisez les codes d'erreur pour l'internationalisation

### Journalisation
- Loggez les erreurs avec un niveau approprié
- Incluez un identifiant de corrélation

## Dépannage

### Erreurs Courantes

#### 400 Bad Request
- Vérifiez le format des données envoyées
- Consultez les `fieldErrors` pour les détails

#### 401 Unauthorized
- Vérifiez le token JWT
- Assurez-vous qu'il n'est pas expiré

#### 403 Forbidden
- Vérifiez les rôles et permissions de l'utilisateur

#### 404 Not Found
- Vérifiez l'URL et les paramètres de la requête

#### 500 Internal Server Error
- Consultez les logs serveur
- Vérifiez la connectivité aux services externes

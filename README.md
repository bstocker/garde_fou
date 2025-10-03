------------------------------------------------------------------------------------------------------
ATELIER GARDE FOU
------------------------------------------------------------------------------------------------------
L’idée en 30 secondes : l'objectif est vérifier qu’un fichier JSON est correctement structuré et utilisable dans une application en passant par un script PHP de vérification (serveur Garde Fou).
  
-------------------------------------------------------------------------------------------------------
Séquence 1 : Le Laboratoire
-------------------------------------------------------------------------------------------------------
Objectif : Création d'une instance dans un laboratoire numérique  
Difficulté : Très facile (~5 minutes)
-------------------------------------------------------------------------------------------------------
RDV sur le laboratoire numérique : <a href="http://lab-boris.fr" target="_blank">Laboratoire</a> **(click droit ouvrir dans un nouvel onglet)** puis créer une instance **+ ADD NEW INSTANCE**

**Astuce pour coller du code dans votre instance du laboratoire**  
Vous aurez à coller du code dans votre instance. Toutefois la combinaison de touche Ctrl+V ne fonctionne pas dans le navigateur.  
La combinaison de touche pour coller du code dans votre instance sera en fonction de votre navigateur.  
- Coller dans Chrome -> Ctrl+Shift+v
- Coller dans Firefox ->  Shift+Insert
- Coller dans EI et Safari -> Ctrl+v
  
---------------------------------------------------
Séquence 2 : Création de votre PHP
---------------------------------------------------
Objectif : Créer un serveur PHP et un script de vérification  
Difficulté : Simple (~10 minutes)
---------------------------------------------------

Dans votre instance du laboratoire copier/coller les codes ci-dessous etape par étape :  

**Installation de l'environnement**  
```
apk add php php-cli php-json php-mbstring curl nano
```
**création d'un fichier validate_json.php**  
```
<?php
// Exemple de validation JSON côté serveur (PHP)

// 1. Récupérer le contenu du JSON (ici en POST)
$payload = file_get_contents("php://input");

// 2. Vérifier taille max (exemple : 2 Mo)
if (strlen($payload) > 2 * 1024 * 1024) {
    http_response_code(400);
    die(json_encode(["error" => "Fichier JSON trop volumineux"]));
}

// 3. Vérifier encodage UTF-8
if (!mb_detect_encoding($payload, 'UTF-8', true)) {
    http_response_code(400);
    die(json_encode(["error" => "Encodage non valide (UTF-8 attendu)"]));
}

// 4. Parser le JSON
$data = json_decode($payload, true);
if (json_last_error() !== JSON_ERROR_NONE) {
    http_response_code(400);
    die(json_encode(["error" => "JSON mal formé"]));
}

// 5. Vérifier les champs obligatoires
$requiredFields = ["videoId", "image360", "video", "labels", "timeline", "version"];

foreach ($requiredFields as $field) {
    if (!isset($data[$field])) {
        http_response_code(400);
        die(json_encode(["error" => "Champ manquant : $field"]));
    }
}

// 6. Vérifier le type des champs principaux
if (!is_string($data["videoId"]) || strlen($data["videoId"]) === 0) {
    die(json_encode(["error" => "videoId doit être une chaîne non vide"]));
}

if (!filter_var($data["image360"]["src"], FILTER_VALIDATE_URL)) {
    die(json_encode(["error" => "image360.src doit être une URL valide"]));
}

if (!filter_var($data["video"]["src"], FILTER_VALIDATE_URL)) {
    die(json_encode(["error" => "video.src doit être une URL valide"]));
}

if (!is_array($data["labels"])) {
    die(json_encode(["error" => "labels doit être un tableau"]));
}

if (!is_array($data["timeline"])) {
    die(json_encode(["error" => "timeline doit être un tableau"]));
}

if (!is_int($data["version"]) || $data["version"] < 1) {
    die(json_encode(["error" => "version doit être un entier ≥ 1"]));
}

// 7. Si tout est OK
http_response_code(200);
echo json_encode(["status" => "OK", "message" => "JSON valide"]);
?>
```
Ctrl+s  
Ctrl+x  
  
**On lance le serveur et réccupérer l'URL de mécanisme de contrôle** 
```
php -S 0.0.0.0:8000
```
Pour récuppérer l'URL de contrôle, cliquez sur le bouton **[OPEN PORT] puis tapez 8000**  
Exemple d'url http://ip10-1-16-4-d3foar1u8201c9uou19g-8000.direct.lab-boris.fr/  
Et votre "API" de contrôle sera donc http://ip10-1-16-4-d3foar1u8201c9uou19g-8000.direct.lab-boris.fr/validate_json.php
  
**Processus de vérification** 
Vous pouvez à present utiliser la solution **Postman** pour envoyer vos structures JSON à votre serveur PHP et demander une vérification de sa part (Si OK alors Code 200 en sortie, Si HS alors code 400 en sortie). N'oubliez pas d'**envoyer vos JSON en POST**.  
  
Exemple de structure JSON Correct :
```
{
  "videoId": "demo123",
  "image360": { "src": "https://cdn.exemple.com/image.jpg" },
  "video": { "src": "https://cdn.exemple.com/video.mp4" },
  "labels": [
    {
      "id": "label1",
      "position": [0, 1.5, -3],
      "content": {
        "label": { "en": "Scalpel", "fr": "Scalpel" },
        "description": { "en": "A surgical tool", "fr": "Un outil chirurgical" }
      }
    }
  ],
  "timeline": [
    { "id": "event1", "type": "marker", "timestamp": 10 }
  ],
  "version": 1
}
```
Exemple de structure JSON Incorrect :
```
{
  "videoId": "demo123",
  "image360": { "src": "https://cdn.exemple.com/image.jpg" },
  "video": { "src": "https://cdn.exemple.com/video.mp4" },
  "labels": [
    {
      "id": "label1",
      "position": [0, 1.5, -3],
      "content": {
        "label": { "en": "Scalpel", "fr": "Scalpel" },
        "description": { "en": "A surgical tool", "fr": "Un outil chirurgical" }
      }
    }
  ],
  "timeline": [
    { "id": "event1", "type": "marker", "timestamp": 10 }
  ],
  "version": 1
}
```
Bravo l'atelier est terminé !
  
---------------------------------------------------
Capitalisation
---------------------------------------------------
Vous avez apris au travers de cet atelier comment monter un serveur PHP qui viendra vérfier la structure d'un JSON. C'est une vérification à faire avant l'execution d'un traitement.

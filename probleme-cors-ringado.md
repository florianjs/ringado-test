# Problème CORS avec le widget Ringado

## Résumé du problème

Le widget Ringado hébergé sur `https://widget.ringado.com/ringado-chat-widget.js?v=2` ne peut pas être chargé à cause d'un problème de configuration CORS côté serveur.

## Erreur observée

```
Blocage d'une requête multiorigine (Cross-Origin Request) : 
la politique « Same Origin » ne permet pas de consulter la ressource distante 
située sur https://widget.ringado.com/ringado-chat-widget.js?v=2. 
Raison : l'en-tête CORS « Access-Control-Allow-Origin » ne correspond pas à « *, * ».
```

## Quel serveur est concerné ?

**Il s'agit du serveur qui héberge le module JavaScript de Ringado** (`widget.ringado.com`), **PAS de votre serveur d'application**.

Le problème se situe au niveau de l'infrastructure de Ringado qui sert le fichier JavaScript du widget.

## Analyse technique du problème

### 1. Configuration CORS invalide

Le serveur `widget.ringado.com` retourne un en-tête CORS malformé :
```
Access-Control-Allow-Origin: *, *
```

Cette valeur est **invalide** selon la spécification CORS. Les valeurs valides sont :
- `*` (autoriser toutes les origines)
- Une origine spécifique : `https://monsite.com`
- Une liste d'origines séparées par des espaces (pas des virgules)

### 2. Problème spécifique aux modules ES6

Le widget utilise la syntaxe ES6 modules (`export`/`import`) qui impose des restrictions CORS plus strictes que les scripts classiques.

## Ce que le serveur Ringado doit corriger

### Solution immédiate (recommandée)

**Modifier la configuration du serveur web** (Apache, Nginx, Cloudflare, etc.) qui sert `widget.ringado.com` pour retourner :

```
Access-Control-Allow-Origin: *
```

Au lieu de :
```
Access-Control-Allow-Origin: *, *
```

### Configuration selon le serveur web

#### Nginx
```nginx
location ~* \.(js)$ {
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods GET,OPTIONS;
    add_header Access-Control-Allow-Headers Content-Type;
}
```

#### Apache
```apache
<FilesMatch "\.(js)$">
    Header set Access-Control-Allow-Origin "*"
    Header set Access-Control-Allow-Methods "GET, OPTIONS"
    Header set Access-Control-Allow-Headers "Content-Type"
</FilesMatch>
```

#### Cloudflare (si utilisé)
Dans le dashboard Cloudflare :
1. Aller dans **Rules** > **Transform Rules**
2. Créer une règle pour modifier les en-têtes de réponse
3. Condition : `hostname eq "widget.ringado.com" and ends_with(uri.path, ".js")`
4. Action : Définir `Access-Control-Allow-Origin` à `*`

### Solution alternative (plus sécurisée)

Si Ringado préfère ne pas autoriser toutes les origines, ils peuvent :

1. **Autoriser les origines courantes** :
```
Access-Control-Allow-Origin: https://*.netlify.app https://*.vercel.app https://*.github.io
```

2. **Implémenter une logique dynamique** pour retourner l'origine de la requête si elle est dans une liste autorisée.

## Solution temporaire côté client

En attendant que Ringado corrige le problème, vous pouvez :

### Option 1 : Charger sans type="module"
```html
<script 
    src="https://widget.ringado.com/ringado-chat-widget.js?v=2"
    operator-id="nn9qoj8kdin25nf">
</script>
```
⚠️ **Risque** : Peut causer des erreurs de syntaxe si le script utilise `export/import`

### Option 2 : Utiliser un proxy CORS
Utiliser un service comme `https://api.allorigins.win/` pour contourner temporairement le problème.

### Option 3 : Héberger une copie locale
Télécharger le fichier JavaScript et l'héberger sur votre propre domaine.

## Conclusion

**Le problème doit être résolu par l'équipe technique de Ringado** en corrigeant la configuration CORS de leur serveur `widget.ringado.com`. 

Il s'agit d'une correction simple qui consiste à remplacer `Access-Control-Allow-Origin: *, *` par `Access-Control-Allow-Origin: *` dans la configuration de leur serveur web.

## Contact recommandé

Contactez le support technique de Ringado en leur transmettant :
1. Cette documentation du problème
2. L'erreur console exacte
3. La demande de correction de l'en-tête CORS malformé

---

*Date du problème : 12 août 2025*  
*Fichier concerné : `https://widget.ringado.com/ringado-chat-widget.js?v=2`*

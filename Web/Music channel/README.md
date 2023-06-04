# Music channel

- Stockage en DB => toutes les requêtes sont préparé + flag situé sur la machine: c'est presque sûr que ce n'est pas une injection.
- Path traversal possible avec le paramètre de l'id de la vidéo youtube: impossible, le basename filtre tout.

- Possibilité de récupérer les vidéo des prochains jour: récupère la vidéo d'aujourd'hui si le header ``X_DATE`` est non spécifié. Il suffit donc d'injecter dans le header ce qu'il faut pour récup n'importe quel jour.
- Stocke le nom de la vidéo dans un fichier sous ``uploads/*`` dans un json avec pour nom de fichier, le paramètre v du lien youtube qu'on a spécifié.
- N'importe quel fichier contenu dans le uploads sont exécutable et affichable : RCE possible.
- Point important, le paramètre v de l'url spécifié est utilisé tel quel pour enregistrer le fichier de cache. Par contre, il est truncate au 11e caractères max. Donc un lien soumis comme: ``https://www.youtube.com/watch?v=dQw4w9WgXcQ.php`` sera requêter pour obtenir le nom de la vidéo avec le lien: ``https://www.youtube.com/watch?v=dQw4w9WgXcQ`` (ca enlève tout ce qu'il y a après)

De là on a tout: RCE dans le cache du fichier: on doit injecter notre code dans un titre de vidéo.
https://youtu.be/kNnHZ8-T0OI contient la RCE. Et donc soumettre: https://www.youtube.com/watch?v=kNnHZ8-T0OI.php
Il suffit ensuite de récupérer la vidéo en changeant la date jusqu'à ce qu'on récupère la bonne (pour créer le fichier de cache).
Se rendre ensuite dans ``uploads/kNnHZ8-T0OI.php`` et apprécié le flag.

*Le char dans la rce est pour bypass le parsing des ``/`` lors du json_encode qui place un ``\`` juste avant.*
Comme YouTube n'accepte pas les caractère type: < > etc... on est obligé de le convertir en html_entities car le code fait un ``html_entity_decode`` juste après avoir récup le titre de la vidéo.

*Voilà voilà pour ca, fuck voltaire si y'a des fautes, je code, je parle pas fr*
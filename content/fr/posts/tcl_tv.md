+++
title = "Ma Smart TV est maintenant une dumb TV"
date = "2025-05-22T10:32:06+01:00"
draft = false
tags = ["tech", "tv"]
translationKey = "tcl-tv"
slug = "ma-smart-tcl-tv-est-maintenant-une-dumb-tv"
+++

...ou au moins, elle se tait et me fiche la paix. 


## TCL et l'enfer des mises à jour.

Depuis peu, je possède une TV TCL miniled de 75 pouces (c89b pour la France et c855 pour le reste de l'Europe) pour environ 3 fois moins cher qu'une Oled de cette taille, le rapport qualité-prix est vraiment excellent.  Mais les Chinois ne sont pas encore au point sur les politiques de mises à jour à priori depuis la sortie du carton en 2024, j'ai eu seulement 1 seule maj: la v412. Si vous écumez un peu les forums vous verrez le bazar, certains pays ont eu la v420, certaines mise à jours donnent des bugs sur certains modèle et pas sur d'autres et enfin, il y a des mises à jour qu'on trouve facilement sur l'internet qui ne rentrent jamais dans le circuit officiel. 

Alors que les firmwares des modèles de 2025 arrivent et sont compatibles avec les télés de 2024, rien n'est proposé pour mettre à jour sur ma TV. La politique de suivi des mises à jour de TCL c'est carrément une blague à ce niveau. Zéro suivi, zéro corrections de bugs une fois la télé officiellement sortie. En l'occurrence ma v412 elle a un soucis de saccades que j'aurais bien aimé voir résolu.  Alors j'ai craqué j'ai choppé la v486 de 2025 et upgrade via usb puis reset d'usine. (Update: Janvier 2026, TCL m'a proposé de mettre à jour le firmware vers une version 5XX qui correspond aux dernièrs modèles.)

Je passe rapidement sur cette histoire parce que finalement ce n'est pas le coeur de l'expérience, mais heureusement que la communauté est là pour monter comment récupérer les bons firmwares pour son modele, la procédure pour l'installer et donner des retours sur les bugs rencontrés parce qu'il n'y a rien côté TCL. Pour les modèles de 2024 toutes les infos sont sur (XDA)[https://xdaforums.com/t/tcl-2024-google-tvs-p655-p755-c655-c655-pro-c765-t8b-c855-115x955.4666345/page-51]


## Configuration de la télé
Si google TV a bien réussi quelque chose c'est bien à faire l'UI la moins user friendly du monde. Recommendations partout pour des services que tu n'as pas, Recommendations pour des programmes dont tu te fiches pas mal, Les applications sont reléguées dans un onglet qui demande plusieurs clics etc..., etc... Sans oublier que l'os de google est plutôt verbeux dans sa communication vers l'extérieur.  Sans rentrer dans les configurations réseaux style Pihole, voyons voir comment on peut rendre cette télé agréable à utiliser au quotidien.  


Maintenant que la télé est à jour au lieu d'être à l'abandon, rendons là plus sympatique à utiliser. 
Tout d'abord il ne faut pas aller rentrer son compte google dans le setup initial.  C'est très important pour plus tard. Rentrer le minimum, faire la configuration de base TCL pour arriver  sur un écran google TV avec quelques appli et c'est tout.  Ensuite il faut se rendre developer et activer le débogage usb afin d'utiliser adb. Enfin, il vous faudra le récupérer les outils pour parler avec Android, notamment "adb" qui se trouve dans le paquet android-tools sous Fedora.

On va commencer par installer un nouveau launcher pour remplacer celui de google: Pitivy Launcher.

```
adb connect <votre ip> 
adb install -r pitivylaucher.apk 
```

Google n'aime pas trop qu'on remplace son launcher et par défaut chaque appui sur la touche home va vous renvoyer vers le launcher par défaut.   
Contournons ça

```
adb shell pm disable-user -k --user 0 com.google.android.setupwraith
```

Maintenant qu'on a mit sous silence le setup google et que le launcher est là vous pouvez aller sur le google play store et ajouter votre compte google. Le fait de faire comme ça, plutôt que de faire le setup google complet permet de ne pas installer toutes les applis googles inutiles: films, games, youtube music, ni les apps partenaires comme alexa.  
A partir de là vous pouvez installer vos applis via le play store, ou passer par adb pour enlever ou désactiver quelques appli. 


```
# Désintaller
adb shell pm uninstall -k --user 0 <app_name>

# Désactiver 
adb shell pm disable-user --user 0 <app_name> 

```

Voici quelques apps qui sont désactivées chez moi: 
- com.mediatek.AirplayAPK
- com.tcl.airplay2
- com.mediatek.airplaydaemon

Il ne reste plus qu'a passer un peu de temps pour configurer notre nouveau launcher et profiter, enfin, d'une interface utile et agréable.

## Bonus: de chouettes fond d'écran
Sur le PlayStore, j'ai téléchargé l'app **Aerial View**. Elle permet de mettre de jolies vidéos 4K HDR en fond d'écran. L'appli fonctionne très bien mais ne se lance pas automatiquement quand la télé doit se mettre en veille. Pour résoudre ce soucis, il faut utiliser l'app **Safety Guard** de TCL. 

- Ouvrir l'app SafetyGuard sur la TV
- Naviguer vers Permission Shield > Auto Launch Permission
- Changer: Auto Manager en haut to Closed - Ca permet de sélectionner manuellement les apps qui peuvent se lancer automatiquement
- Scroller jusque Aerial View et le changer en Ouvert

Voilà nos (beaux) écrans de veille sont maintenant fonctionnels.


### Liens utiles
(Guide de survie du tcliste)[https://docs.google.com/document/d/1sAAPpFFwf1YozScXeZhaVYTczKc9XuOsNNBfcLM3gFg/edit?tab=t.0#heading=h.kgd99a7novk6]  en français
(XDA forums)[https://xdaforums.com/t/tcl-2024-google-tvs-p655-p755-c655-c655-pro-c765-t8b-c855-115x955.4666345/page-51]

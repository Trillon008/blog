+++
title = "Headless streaming with bazzite"
date = "2025-05-22T10:32:06+01:00"
draft = false
tags = ["tech", "linux"]
translationKey = "streaming_with_bazzite"
slug = "headless-streaming-with-bazzite"
+++

Ou n'importe quelle autre distro

J'ai récemment craqué et décidé d'acheter une 5070Ti en replacement de ma RX5700XT qui doucement, prend de l'âge.  L'hésitation fu longue tant les prix prohibitifs des cartes graphiques me refroidissent, mais j'ai fini par craquer. 
Honnetement j'aurais préféré prendre une carte AMD, mais le soucis avec le HDMI 2.1 non supporté sur le driver opensource était trop problématique et m'a forcé chez les verts. Le PC étant branché sur ma TV pour une expérience console, le Display Port n'était pas une option. La qualité aléatoire des adaptaeurs DP -> HDMI à fini de me convaincre d'aller chez Nvidia. 


## HDMI 2.1 issues 
Trop souvent passé sous silence, ce bug (ou plutôt cette absence de fonctionalité) est bien connue des joueurs Linux qui essaient de profiter des capacités de leurs TV 4K comme le HDR et le VRR. Les télés n'ont pas de ports DP et seule la norme HDMI 2.1 permet de jouer en 4K à 120 fps ou encore d'utilier le VRR (Variable Refresh Rate). Ce défaut n'est pas du à AMD qui ne fait pas l'effort de faire un driver correct, mais bien au Consortium HDMI qui ne veut pas voir de driver Open Source de leur Norme HDMI 2.1 .  
Tant d'efforts de la part des ayants droits pour essayer de protéger leurs license alors que la norme HDMI est un gruyère absolu. Merci aussi aux fabricants de TV qui offrent de plus en plus de fonctions pour le jeu sur TV mais qui préfèrent ne pas mettre d'entrée DisplayPort. Entrée qui n'est soumise à aucune license et qui coute donc juste, quelques centimes. 


## L'idée
Ce type de setup d'ordinaire dans le sens PC du bureau vers PC de la télé, mais moi je l'utilise dans l'autre sens, le PC à côté de ma télé stream vers le PC de mon bureau ou vers le Steam Deck.
Pour diverses raisons, ce setup me convient mieux.
Dans l'absolu cela ne change pas le fait que j'ai besoin d'un setup Headless.


## Headless streaming what is it
Sur Steam, quand vous streamez vos jeux, basiquement, votre écran principal, reste allumé. Votre autre écran streamé est aussi allumé et vous vous retrouvez dans une solution avec deux écrans allumés. Et donc, l'écran numéro 1 n'est pas utilisable par une autre personne qui voudrait utiliser le PC. Vous avez donc un miroir parfait des écrans en temps quasi réel.  
Dans notre setup headless, on veut que notre écran source reste éteint pendant que l'écran qui reçoit le stream est forcément allumé. Comme si vous aviez un serveur dans écran. 


## Comment faire
- Récuprer l'edid de son écran source
Nous allons créer un écran virtuel. Pour ça il nous faut un EDID d'écran, on peut en trouver sur internet, dans mon cas je vais simplement prendre celui de mon PC qui est un écran 2K - 144hz. 

```
## Trouver les ports écran utilisé
$ for p in /sys/class/drm/*/status; do con=${p%/status}; echo -n "${con#*/card?-}: "; echo "$p: $(cat $p)"; done

DP-1: /sys/class/drm/card1-DP-1/status: connected
DP-2: /sys/class/drm/card1-DP-2/status: connected
DP-3: /sys/class/drm/card1-DP-3/status: disconnected
HDMI-A-1: /sys/class/drm/card1-HDMI-A-1/status: disconnected

## Get the edid file from Diplay Port 1
$ cat /sys/class/drm/card1-DP-1/edid  > /tmp/vscreen

## check edid is not empty
$ hexdump -c /tmp/vscreen | head
```

- Trouver un emplacement HDMI/DP libre sur votre écran destination
Pour ça: 
```
for p in /sys/class/drm/*/status; do con=${p%/status}; echo -n "${con#*/card?-}: "; echo "$p": $(cat $p)"; done
```
Pour moi HDMI-1-A est occupé, c'est la télé. Je vais prendre DP-1 comme port disponible pour mon nouvel écran virtuel. 

- Copier l'edid file sur votre destination et le mettre dans un dossier dédié. Personnellement j'ai copié le fichier via ssh et je l'ai mit dans **/lib/firmware/edid/**  
Ensuite il nous faut ajouter ce nouvel écran via un argument kernel. Sur bazzite je conseille la commande rpm-ostree kargs --editor  qui permet de modifier via son editeur préféré. Pour l'exemple, j'ai utilisé un append qui marche tout aussi bien.  

```
rpm-ostree kargs append-if-missing="firmware_class.path=/lib/firmware/edid drm.edid_firmware=DP-1:vscreen video=DP-1:2560x1440@144e"
```
Je n'explique pas tous les arguments ça se comprend assez bien.  
Mais... c'est très important de bien mettre la résolution et le rafraichissement surtout si comme moi, votre écran ne s'est jamais allumé. Honnetement, je n'ai pas bien compris pourquoi, mais si la résolution n'est pas mise je suis obligé d'allumer et éteindre l'écran. Comme pour initier l'écran. Bref

Et voilà, tout devrait fonctionner, il reste a bidouiller les paramètres de sunshine et moonlight pour trouver le bon équilibre. 
Dans sunshine vous pouvez lancer certaines commandes arbitraires en début et en fin de stream. C'est possible par exemple d'eteindre un écran et d'en allumer un autre. par exemple:
```
## Eteind l'écran branché sur HDMI-1 et allume celui sur Display Port 2
/usr/bin/kscreen-doctor output.HDMI-A-1.disable && /usr/bin/kscreen-doctor output.DP-2.enable
```
Dans mon cas ce n'est pas nécessaire, mais ça peut servir. 
Enjoy :)

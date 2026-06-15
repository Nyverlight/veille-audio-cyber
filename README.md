# Veille cybersécurité audio — mise en place pas-à-pas (Routines cloud)

Ce guide vous accompagne de zéro jusqu'à un épisode audio qui arrive seul
chaque matin dans votre application de podcast, **sans laisser votre PC
allumé**. Tout le traitement tourne sur le cloud d'Anthropic ; le stockage et
la diffusion passent par un dépôt GitHub + GitHub Pages (gratuits).

---

## A. Ce que fait la routine chaque matin

1. Installe ses dépendances dans son bac à sable cloud.
2. Récupère les actualités cybersécurité des dernières ~30 h depuis vos flux
   RSS (`fetch_news.py`), avec un dédoublonnage grossier.
3. **Claude lui-même** fusionne les doublons restants, traduit l'anglais,
   hiérarchise (vulnérabilités activement exploitées, incidents majeurs,
   tendances) et rédige un script parlé en français.
4. Transforme ce script en MP3 avec une voix neuronale française
   (`make_audio.py` / edge-tts, gratuit).
5. Met à jour le flux podcast `feed.xml` et applique une rétention
   (`update_feed.py`).
6. Commite et pousse le tout sur GitHub → publié via GitHub Pages.

Vous, le matin : vous ouvrez votre app podcast, l'épisode du jour est là.

---

## B. Prérequis

- **Un plan Claude payant** (Pro suffit) avec accès à Claude Code sur le web
  (`claude.ai/code`).
- **Un compte GitHub** (gratuit).
- C'est tout. Aucune installation sur votre machine n'est nécessaire.

---

## C. Étape 1 — Créer le dépôt et y déposer le kit

1. Sur GitHub, créez un dépôt **public** nommé `veille-audio-cyber`.
   (Public = GitHub Pages gratuit. Le contenu n'est que du résumé d'actu
   cyber ; l'URL reste de fait « non listée ». Si vous voulez un dépôt privé,
   Pages nécessite un plan GitHub payant — voir la section Dépannage.)
2. Déposez-y tous les fichiers de ce kit, en conservant l'arborescence :
   ```
   veille-audio-cyber/
     config/config.json
     config/sources_cyber.txt
     scripts/fetch_news.py
     scripts/make_audio.py
     scripts/update_feed.py
     requirements.txt
     setup.sh
     ROUTINE_PROMPT.md
     README.md
     .gitignore
     feed.xml
     audio/.gitkeep
   ```

⏱️ ~10 min

---

## D. Étape 2 — Activer GitHub Pages

1. Dépôt → **Settings** → **Pages**.
2. Source : **Deploy from a branch**, branche **main**, dossier **/(root)**.
3. Enregistrez. GitHub affiche l'URL de votre site, du type
   `https://VOTRE-USER.github.io/veille-audio-cyber`.
4. Ouvrez `config/config.json` et remplacez la valeur de `base_url` par cette
   URL **exacte** (sans slash final). Faites de même dans `feed.xml`
   (`<link>` et `<itunes:image href>`).

⏱️ ~10 min (+ quelques minutes de propagation côté GitHub)

---

## E. Étape 3 — (Optionnel) image de couverture

Déposez un fichier `cover.jpg` (carré, ex. 1400×1400) à la racine du dépôt :
il servira de vignette dans l'app podcast. Sans lui, le flux reste valide.

---

## F. Étape 4 — Personnaliser la configuration

Dans `config/config.json` :
- **`voice`** : voix de synthèse. Réglée ici sur
  `fr-FR-RemyMultilingualNeural` (multilingue) : c'est volontaire, car la
  cyber est dense en termes anglais (ransomware, threat actor, noms
  d'éditeurs…) que les voix multilingues prononcent mieux. Alternatives :
  `fr-FR-VivienneMultilingualNeural` (féminine, multilingue),
  `fr-FR-HenriNeural` / `fr-FR-DeniseNeural` (mono-langue).
  Pour lister les voix disponibles : `edge-tts --list-voices`.
- **`rate`** : débit de lecture, ex. `+10%` pour accélérer.
- **`lookback_hours`** : fenêtre temporelle (30 h par défaut).
- **`max_items_fetch`** : nombre max d'items bruts transmis à l'éditorial
  (80 ici, car la cyber a beaucoup de sources et un fort volume).
- **`keep_episodes`** : nombre d'épisodes conservés (30 par défaut).
- **`feed_title` / `feed_description`** : titre et description du podcast.

Dans `config/sources_cyber.txt` : ajoutez/retirez des flux RSS. Plusieurs
flux de presse FR (LeMagIT, Le Monde Informatique, Silicon) y sont en mode
« à confirmer » : le premier test (étape 6) vous dira lesquels répondent et
lesquels sont morts.

⏱️ ~15-20 min

---

## G. Étape 5 — Créer la routine

Allez sur `claude.ai/code/routines` → **Create**.

1. **Dépôt** : sélectionnez votre dépôt `veille-audio-cyber`.
2. **Pushes de branche** : activez **« Allow unrestricted branch pushes »**
   pour ce dépôt, afin que la routine puisse commiter sur `main` (que Pages
   sert). Sans cela, Claude ne pousse que sur des branches `claude/...`.
3. **Modèle** : choisissez **Opus** (qualité du tri éditorial).
4. **Environnement réseau** — étape critique. Le bac à sable a un accès réseau
   restreint ; autorisez explicitement :
   - `pypi.org`, `files.pythonhosted.org` (installation des dépendances),
   - `github.com` (push),
   - **le service de synthèse vocale** : `speech.platform.bing.com`
     (utilisé par edge-tts ; à confirmer, le domaine peut évoluer),
   - **tous les domaines de vos sources cyber**, par exemple :
     cert.ssi.gouv.fr, cisa.gov, zataz.com, it-connect.fr, lemagit.fr,
     lemondeinformatique.fr, 01net.com, thehackernews.com, feedburner.com,
     bleepingcomputer.com, krebsonsecurity.com, darkreading.com,
     securityweek.com, therecord.media, grahamcluley.com, schneier.com,
     theregister.com, securelist.com, paloaltonetworks.com, microsoft.com,
     googleprojectzero.blogspot.com, isc.sans.edu
     (+ leurs sous-domaines/CDN si besoin).
5. **Script de setup** : `bash setup.sh` (ou laissez la routine l'exécuter via
   le prompt — il est déjà prévu à l'étape 1 du prompt).
6. **Trigger** : type **schedule**, fréquence **quotidienne**, heure ex. 05:30.
7. **Prompt** : collez l'intégralité du contenu de `ROUTINE_PROMPT.md`.
8. Créez la routine.

⏱️ ~20-30 min

---

## H. Étape 6 — Test « Run now » et vérifications

Lancez la routine manuellement avec **Run now**, puis vérifiez, dans l'ordre :

1. **Setup** : `setup.sh` a installé les dépendances sans erreur.
2. **Collecte** : le log de `fetch_news.py` liste vos flux. Repérez les
   `[ECHEC]` → soit l'URL est morte (corrigez `sources_cyber.txt`), soit le
   domaine n'est pas autorisé (revoyez l'allow-list réseau, étape G.4).
   Le message « aucune actualité récupérée » = problème réseau ou flux morts.
   C'est aussi le moment de **valider les flux FR « à confirmer »** et de
   décider si vous activez le flux `avis` du CERT-FR (gros volume de CVE).
3. **Éditorial** : ouvrez `work/script.txt` (dans la session) — le texte
   doit être du français parlé propre, sans markdown ni URL, avec les
   vulnérabilités regroupées plutôt que listées une par une.
4. **Audio** : `audio/cyber-AAAA-MM-JJ.mp3` est créé et fait plus de
   quelques Ko. Si échec → l'allow-list ne couvre probablement pas
   `speech.platform.bing.com`.
5. **Flux** : `feed.xml` contient un nouvel `<item>` avec la bonne
   `<enclosure>`.
6. **Publication** : un commit « Veille Cyber - … » est poussé sur `main`.
7. **Modèle** : confirmez que la session a tourné sur **Opus** (et non un
   repli sur Sonnet) — cela a parfois été observé sur les tâches planifiées.

⏱️ ~30-60 min (l'essentiel du calibrage : longueur, style, voix, tri des CVE)

---

## I. Étape 7 — S'abonner dans l'app podcast

1. Vérifiez que `https://VOTRE-USER.github.io/veille-audio-cyber/feed.xml`
   s'ouvre bien dans un navigateur (XML affiché).
2. Dans votre app podcast (Pocket Casts, Apple Podcasts, AntennaPod, etc.) :
   « Ajouter par URL » / « Add by URL » → collez l'URL du `feed.xml`.
3. L'épisode de test apparaît. Les jours suivants, chaque nouvel épisode
   arrivera automatiquement.

⏱️ ~10 min

---

## J. Dépannage

- **Aucune actu récupérée** : 90 % du temps, un domaine de source manque dans
  l'allow-list réseau, ou une URL de flux est morte. Le log de `fetch_news.py`
  pointe le coupable.
- **Pas de MP3 généré** : `speech.platform.bing.com` n'est pas autorisé, ou son
  domaine a changé — vérifiez et mettez à jour l'allow-list.
- **Épisode trop long / noyé sous les CVE** : c'est le piège classique de la
  veille cyber. Activez l'allow-list mais gardez le flux `avis` du CERT-FR
  commenté, et faites confiance à l'instruction de regroupement des
  vulnérabilités dans `ROUTINE_PROMPT.md` (étape 3). Réduisez aussi
  `max_items_fetch` si nécessaire.
- **L'app podcast ne voit pas le flux** : Pages pas encore propagé (attendez
  quelques minutes), ou `base_url` mal renseigné dans `config.json` (doit être
  l'URL Pages exacte, sans slash final).
- **La session tourne sur Sonnet malgré Opus** : re-sélectionnez le modèle ;
  si le souci persiste, c'est une limite connue de la preview à signaler.
- **Dépôt privé voulu** : GitHub Pages sur dépôt privé exige un plan GitHub
  payant. Alternative gratuite : garder le dépôt privé et faire livrer le MP3
  autrement (dossier synchronisé, e-mail, Telegram) — au prix de l'expérience
  « vrai podcast ».

---

## K. Faire cohabiter plusieurs domaines (IA + cyber)

Le moteur (`fetch_news.py`, `make_audio.py`, `update_feed.py`) est générique :
il ne dépend que de `config.json`. Si vous avez déjà la veille IA, ce dépôt
cyber est simplement un **second dépôt + une seconde routine** indépendants,
avec leur propre `base_url`, leur propre `sources_cyber.txt` et le préfixe
audio `cyber-`. Rien n'est partagé : les deux podcasts vivent côte à côte dans
votre app, chacun avec son flux.

Pour ajouter encore un autre domaine plus tard, dupliquez le même schéma
(nouveau dépôt ou nouveau couple config/sources + nouvelle routine).

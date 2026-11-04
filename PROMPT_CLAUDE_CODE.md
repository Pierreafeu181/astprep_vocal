# Prompt Claude Code : prototype « Jury IA » dans un clone du front ASTPrep (via le connecteur Lovable)

> Colle tout ce qui suit dans Claude Code, lancé à la racine du repo `astprep_vocal`, dans lequel tu auras copié le dossier `docs/` de ce kit (captures + brief).

---

## 🎯 Mission

Tu vas piloter **Lovable** via son connecteur MCP pour construire un **prototype privé** destiné au CEO d'ASTPrep, une plateforme française de préparation au TAGE MAGE et aux admissions parallèles en école de commerce. Le prototype a deux parties :

1. **Un clone visuel quasi identique du front ASTPrep** : layout, couleurs, typographie, boutons, cartes, sidebar. Quand le CEO l'ouvre, il doit avoir l'impression d'être sur **sa** plateforme.
2. **Une nouvelle fonctionnalité, « Jury IA »** : un simulateur vocal d'entretien d'oral AST, avec un jury IA par école (HEC, ESSEC, ESCP) et un débrief noté après l'entretien. Elle doit avoir l'air **déjà intégrée nativement** à la plateforme.

Contexte complet (marché, choix de la feature, agents ElevenLabs existants, grille de débrief) : **lis d'abord `docs/BRIEF_simulateur_oral_ASTPrep.md`**.
Références visuelles : **regarde TOUTES les captures dans `docs/screens/`** avant d'écrire le moindre prompt Lovable. Elles sont la source de vérité du design.

---

## 0. Règles de travail (à respecter tout du long)

- **Plan d'abord.** Avant d'appeler Lovable, propose-moi un plan court : l'ordre des pages, ce que tu enverras à Lovable à chaque étape, et une estimation du nombre de messages. Attends mon OK. Chaque `send_message` consomme mes crédits Lovable.
- **Un sujet par message Lovable.** Un message pour le design system et le layout, puis un message par page. Pas de méga-prompt unique.
- **Joins les captures** correspondantes à chaque message Lovable (upload via `get_file_upload_url`).
- **Vérifie après chaque étape** :
  - fais un `get_diff` sur le message ;
  - ouvre la `preview_url` et fais une capture avec Playwright (Chromium est installé) au même format que la référence, 2000 px de large ;
  - compare avec la capture d'origine ;
  - liste les écarts (espacements, rayons, ombres, tailles de police, couleurs) et envoie un message correctif **ciblé**.
  - Vise le « pixel-perfect » sur le Dashboard, la sidebar et les pages Jury IA.
- Si Lovable se met en pause sur une approbation (`awaiting_input`), **montre-moi** l'outil et ses paramètres et attends ma décision.
- **Projet privé.** Mets la visibilité du projet Lovable en privé dès la création.
- Résume-moi ce qui est fait à la fin de chaque étape (2 ou 3 lignes + la preview).

---

## 1. Mise en place

1. `list_workspaces`, puis choisis le workspace (demande-moi s'il y en a plusieurs).
2. `create_project` avec un message initial qui pose la stack et le design system (section 2), sans les pages. Puis `render_project_widget`.
3. Passe le projet en privé (`set_project_visibility`).
4. Avec `set_project_knowledge`, enregistre de façon persistante le design system (section 2), les règles de contenu (section 5) et les garde-fous (section 6). Toutes les itérations Lovable doivent les respecter.
5. **GitHub.** Mon repo est https://github.com/Pierreafeu181/astprep_vocal. Vérifie si le connecteur permet de lier le projet Lovable à un repo GitHub **existant**.
   - Si c'est le cas, fais-le.
   - Sinon, dis-le-moi clairement. La liaison se fait alors dans l'éditeur Lovable (Settings → GitHub), et Lovable crée en général un **nouveau** repo. Propose-moi la meilleure option : utiliser le repo créé par Lovable et garder `astprep_vocal` pour les docs, ou y pousser ensuite.
   - **N'écrase rien** dans mon repo sans me demander.

Stack Lovable : la stack par défaut (React + Vite + TypeScript + Tailwind + shadcn/ui, Lovable Cloud / Supabase pour le backend). Routing côté client avec une route par page.

---

## 2. Design system (relevé sur les captures, à appliquer partout)

> Les couleurs viennent d'un échantillonnage des pixels des captures. Elles peuvent dévier de quelques unités : arrondis-les si un token Tailwind proche tombe juste, mais **ne change pas la teinte**.

### Couleurs
| Token | Valeur | Usage |
|---|---|---|
| `primary` | `#865CF5` | Chiffres clés (328, 373/600…), onglet actif, soulignement d'onglet, liens, avatar, points |
| `primary-btn` | dégradé `#8B5CF6` → `#6D45E3` (relevé `#7A50EC` / `#774DE6`) | Boutons pleins (« Lancer la session », « Voir la correction »…) avec une **ombre violette diffuse** dessous |
| `primary-soft` | `#EBE2FD` / `#F3EEFE` | Fond des pastilles d'icône, chips sélectionnées douces, badge « Bientôt » (`#E8DEFC`) |
| `ink` | `#0B021D` | Titres et texte principal (quasi noir violacé) |
| `muted` | `#696476` | Sous-titres, métadonnées (« 15 questions · 20 minutes ») |
| `dark-panel` | `#332753` (haut légèrement plus clair, `#3F375D`) | Panneaux sombres : « Tes meilleurs scores », bannière « Recommandé pour toi », carte « Ton Discord privé », carte « Ta session » |
| `dark-row` | `#413561` | Lignes de classement dans le panneau sombre |
| `success-bg` / `success` | `#E3F2DD` / `#3C5A32` | Badges « Passé », « Fait », « À jour », « +40 depuis ton premier blanc » |
| `live` | `#C15750` sur fond rose pâle | Badge « En direct » |
| Sous-tests | Compréhension `#865CF5`, Calcul `#6F8FE8`, Raisonnement `#65B2A6`, Conditions min. `#D48D4C`, Expression `#C1598A`, Logique `#8E5FB2` | Points de couleur et badges de catégorie (texte coloré sur fond blanc, bordure fine) |
| Fond d'app | dégradé très doux : lavande `#EFEBFD` en haut à gauche, bleu-lavande `#E6E8FE` au centre, rosé `#EFE4EE` en bas à droite | `body`. Un **léger grain ou une trame de points** est visible au zoom ; reproduis-le subtilement si possible |
| Sidebar | `#F9F4FE` translucide | Panneau gauche |

### Surfaces et formes
- **Cartes** : blanc translucide (environ `rgba(255,255,255,0.65)`), `backdrop-blur`, **bordure blanche 1 px**, rayon d'environ **24 px**, ombre très douce et large (teinte violette, faible opacité).
- **Sidebar** : carte verticale flottante, rayon d'environ 28 px, marge autour, pleine hauteur.
- **Topbar** : carte flottante horizontale de même style, avec à gauche le titre de la page courante (texte moyen) et à droite :
  - une barre de recherche « Rechercher un cours, un exercice, une notion.. » avec badge `⌘K` ;
  - une icône cloche ;
  - une icône lune (mode sombre).
- **Boutons primaires** : pilule (rayon complet), dégradé violet, texte blanc medium, icône à gauche (▷ lecture, ⚡, ↗…), ombre violette.
- **Boutons secondaires** : pilule blanche, texte `ink` ou `primary`, bordure fine (« Refaire le sous-test », « Relancer »).
- **Chips de sélection** (2 min / 5 min / 10 min, Facile / Moyen / Difficile) : pilules blanches à texte `primary` ; la sélection est pleine en violet avec texte blanc et ombre.
- **Onglets** (Compréhension, Calcul…) : icône + libellé + compteur gris en pastille. L'actif est en `primary` avec soulignement violet de 2 px ; les inactifs sont sur un fond légèrement grisé.
- **Badges de statut** : petites pilules (« Pas fait » blanc/gris, « Fait » vert avec point, « Passé » vert avec ✓, « Recommandé » plein violet).
- **Pastille d'icône** : carré arrondi d'environ 44 px, fond `primary-soft`, petit point ou icône violette au centre.
- **Barres de progression** : piste `#ECE7F4`, remplissage violet en dégradé, rayon complet, environ 6 px de haut.
- **Icônes** : style trait fin type **lucide** (déjà dans shadcn).

### Typographie
- Police sans-serif géométrique arrondie, **proche de Poppins**. Par défaut, utilise Poppins (Google Fonts) en 400/500/600.
- Si tu peux vérifier la vraie police (Chrome DevTools sur app.astprep.com → font-family calculée), dis-le-moi et on l'ajuste.
- **Titre de page** : environ 40 px, 500, `ink`, interlettrage légèrement serré, **qui finit toujours par un point** (« Tes statistiques. », « Travaille les sous-tests. »).
- **Sous-titre de page** : environ 17 px, `muted`.
- **Grands chiffres** : environ 40 px, `primary`, poids léger à moyen, suivis de « / 600 » ou « / 60 » en petit gris.
- **Labels de section dans la sidebar** (Apprendre, S'entraîner, Progresser) : petits, gris, interlettrage légèrement large.

### Sidebar (structure exacte, dans cet ordre)
- **Logo** : pastille ronde dégradée violette avec un triangle « A » blanc, puis « ASTPrep ». Bouton « ‹ » à droite pour replier.
- **Sélecteur d'examen** : carte avec pastille « T » et « TAGE MAGE » en majuscules medium, chevron.
- **Navigation** :
  - Dashboard (maison), Parrainage (cadeau)
  - **Apprendre** : Parcours (cible), Cours (livre), Cours en direct (play cerclé)
  - **S'entraîner** : Sous-tests (calques), TAGE MAGE Blanc (presse-papier), Générateur (étincelles), **Jury IA** (micro), avec un badge pilule « Nouveau » violet plein. **C'est la seule entrée ajoutée.**
  - **Progresser** : Statistiques (courbe), Carnet d'erreurs (signet), Outils (grille, dépliable) avec Flashcards, Calcul mental, Lecture alphabétique
- **Item actif** : carte blanche pleine à l'intérieur de la sidebar, texte `ink`, légère ombre.
- **Bas de la sidebar** : carte utilisateur avec avatar rond violet « BP » et un petit chevron, « Baptiste Puiferrat » / « Tage Mage ». Au clic, un menu flottant (capture 01) : nom + email tronqué, puis Mon abonnement, Paramètres, Gérer mes cookies, Notifications et nouveautés.

---

## 3. Pages à cloner (par ordre de priorité)

Toutes les données sont **fictives et codées en dur** (fichier `src/data/mock.ts`). Aucun backend pour ces pages.

**P1, pixel-perfect (le CEO les verra) :**
1. **Dashboard** (capture 14) :
   - « Bonjour, Baptiste. » ;
   - carte Estimation 328/600 avec badge +40 et courbe ;
   - carte Série avec mini-heatmap ;
   - carte Carnet d'erreurs ;
   - Programme du jour (liste et bouton « Lancer ma session ») ;
   - Score par sous-test ;
   - carte sombre « Ton Discord privé ».
   - **Ajoute** une petite carte ou bannière d'annonce dans le style des panneaux sombres, sans casser la grille : « Nouveau · Jury IA » avec le texte « Entraîne-toi à l'oral HEC, ESSEC et ESCP avec un jury IA. » et un bouton « Essayer ».
2. **Statistiques** (capture 06).
3. **Sous-tests** (capture 09).

**P2, fidèles mais sans obsession :**
4. TAGE MAGE Blanc (08)
5. Générateur (07)
6. Parcours (12)
7. Cours (11)
8. Cours en direct (10)
9. Parrainage (13)
10. Carnet d'erreurs (05)
11. Outils : Flashcards (04), Calcul mental (03), Lecture alphabétique (02)

Les boutons de ces pages ne font rien (ou affichent un toast « Prototype »). Seul **Jury IA** est fonctionnel.

**Classements fictifs** : n'utilise **pas** les prénoms d'élèves visibles sur les captures. Invente des initiales et prénoms neutres.

---

## 4. La nouvelle fonctionnalité : « Jury IA »

> **Note produit.** La capture Parrainage montre qu'ASTPrep vend déjà une « préparation aux oraux » (valeur 127 €). Jury IA doit donc se présenter comme **le complément d'entraînement illimité** de cette préparation, pas comme son remplaçant. Glisse cette idée dans les textes.

- **Nom** : **Jury IA**. Accroche de page : « Passe ton oral avant le jour J. »
- **Route** : `/jury-ia`, rangée dans la sidebar sous **S'entraîner**.
- **Ton** : tutoiement côté interface, comme le reste de la plateforme. Le jury, lui, vouvoie pendant l'entretien.

### 4.1 Page d'accueil `/jury-ia`
Même structure que « Calcul mental » / « Lecture alphabétique » (colonne principale + panneau sombre à droite) :

- **En-tête** :
  - titre : « Passe ton oral avant le jour J. » ;
  - sous-titre : « Choisis ton école, entraîne-toi face à un jury IA qui connaît son format, et reçois un débrief noté juste après. »
- **Carte « Choisis ton école »**, avec trois grandes cartes cliquables (HEC, ESSEC, ESCP) disposées comme les cartes d'opération du Calcul mental (sélection = bordure violette + ✓). Chaque carte contient :
  - un **monogramme** dans une pastille (« HEC », « ESSEC », « ESCP » en texte), **pas de logo officiel** ;
  - le nom complet (HEC Paris, ESSEC Business School, ESCP Business School) ;
  - une ligne de métadonnées (format, durée, langue) lue dans la config ;
  - un badge de statut : « Prêt » (vert) si un `agentId` est renseigné, « Bientôt » (lavande) sinon.
- **Carte « Niveau du jury »** : chips Bienveillant / Neutre / Exigeant (Neutre par défaut).
- **Carte « Ce que le jury va évaluer »**, dans le style de « Pourquoi ça compte » : les 5 critères de l'école sélectionnée, sous forme de petites pastilles.
- **Bouton primaire en haut à droite** : « ▷ Lancer l'entretien ». Il est désactivé si l'école n'a pas d'agent.
- **Panneau sombre à droite** :
  - « Tes dernières simulations » : liste façon classement avec école, date et note /20 ;
  - « Meilleure note » avec un gros chiffre ;
  - état vide : « Aucune simulation pour l'instant. Lance ton premier entretien pour obtenir ton débrief. »
- **Bandeau discret de consentement** au-dessus du bouton, ou dans une modale au lancement : « L'entretien est transcrit pour générer ton débrief. Ta voix n'est pas conservée par ASTPrep. » Case à cocher obligatoire.

### 4.2 Page entretien `/jury-ia/:school/session`
- **Topbar** : « Jury IA · ESSEC ».
- **Au centre**, une grande carte glass avec un **orbe animé** (dégradé violet, pulsation pilotée par le niveau audio) et deux états lisibles : « Le jury parle… » et « À toi ». S'y ajoutent un chrono (mm:ss), le nom du jury (« Jury ESSEC · Admission sur titre ») et le niveau choisi.
- **À droite**, un panneau sombre « Transcription » qui se remplit en direct (messages jury / toi), repliable.
- **En bas** : « Couper le micro » (secondaire) et « Terminer l'entretien » (primaire).
- **Technique** :
  - SDK `@elevenlabs/react`, hook `useConversation` ; `startSession({ agentId })` après la demande de permission micro ;
  - récupère le `conversationId` renvoyé ;
  - à la fin, redirige vers le débrief.
  - Les agents de démo sont **publics** : aucun secret côté front pour la conversation. Vérifie l'API exacte du SDK dans la doc actuelle d'ElevenLabs, ne devine pas.

### 4.3 Page débrief `/jury-ia/:school/debrief/:conversationId`
- **Titre** : « Ton débrief est prêt. » ; **sous-titre** : école, date et durée.
- **Ligne de cartes**, comme le haut des Statistiques :
  - grande **note /20** en violet, avec un badge vert « +X depuis ta dernière simu » s'il y a un historique ;
  - **radar** des 5 critères (recharts, violet) ;
  - **5 barres** critère par critère (/4), dans le style « Profil par sous-test ».
- **Carte « Tes points forts »** : liste avec citations en italique.
- **Carte « 3 moments clés »** : pour chacun, la question du jury, ce que tu as répondu, pourquoi ça coince et une **réponse réécrite** (encadré `primary-soft`).
- **Transcription complète** dans un accordéon.
- **Actions** : « Refaire un entretien » (primaire) et « Retour à Jury IA ».
- **Pendant l'analyse** : un état de chargement élégant (« Le jury délibère… », skeletons). L'analyse ElevenLabs prend quelques secondes à une minute.

### 4.4 Backend du débrief
- **Edge function** `get-debrief` (Lovable Cloud / Supabase) qui appelle l'API ElevenLabs pour récupérer la conversation (`GET /v1/convai/conversations/{conversation_id}`), avec la clé en **secret** `ELEVENLABS_API_KEY`, jamais dans le front.
- **Côté front** : polling toutes les 3 s jusqu'à ce que l'analyse soit terminée (maximum 2 min).
- **Mapping** à partir de `analysis.evaluation_criteria_results` (scores sur 4) et de `analysis.data_collection_results`, qui contient :
  - `note_sur_20` ;
  - `debrief_points_forts` ;
  - `debrief_axes_amelioration`.
- **Vérifie la forme réelle** de la réponse dans la doc ou avec un vrai appel avant de mapper.
- **Mode démo** : si `?demo=1` est présent dans l'URL, ou si l'appel échoue, affiche un débrief fictif crédible issu de `mock.ts`. La démo ne doit jamais planter devant le CEO.
- **Historique** : stocke les simulations terminées (école, date, note, conversationId) en `localStorage`, dans un try/catch, pour alimenter « Tes dernières simulations ». Pas de base de données pour l'instant.
- Quand tu arrives à cette étape, **demande-moi la clé API ElevenLabs** : je la saisirai moi-même dans les secrets Lovable, ne me la fais pas coller dans le chat.

### 4.5 Configuration des écoles (je la complèterai moi-même)
Crée `src/config/schools.ts`, typé, qui est la **seule** source de vérité. Les fiches HEC et ESCP restent **à compléter**, n'invente rien :

```ts
export type SchoolId = "hec" | "essec" | "escp";

export const SCHOOLS: Record<SchoolId, School> = {
  hec: {
    id: "hec",
    shortName: "HEC",
    fullName: "HEC Paris",
    agentId: "", // TODO Baptiste : agent ElevenLabs à créer
    format: "À compléter",
    duration: "À compléter",
    language: "À compléter",
    criteria: [/* TODO : 5 critères */],
  },
  essec: {
    id: "essec",
    shortName: "ESSEC",
    fullName: "ESSEC Business School",
    agentId: "agent_4001m6k72ymffqnscw4vykmqxj30", // agent existant
    format: "Entretien de motivation et de personnalité",
    duration: "≈ 15 min (simulation) · ≈ 45 min le jour J",
    language: "Français",
    criteria: [
      "Structure et clarté",
      "Cohérence du projet",
      "Connaissance de l'école",
      "Aisance orale",
      "Questions difficiles",
    ],
  },
  escp: {
    id: "escp",
    shortName: "ESCP",
    fullName: "ESCP Business School",
    agentId: "", // TODO Baptiste
    format: "À compléter",
    duration: "À compléter",
    language: "À compléter",
    criteria: [/* TODO */],
  },
};
```

Les identifiants des critères ESSEC dans l'analyse ElevenLabs sont : `structure_clarte`, `coherence_projet`, `connaissance_ecole`, `aisance_orale`, `questions_difficiles`. Fais correspondre chaque libellé à son identifiant dans la config.

Ajoute aussi un `README` qui explique comment ajouter une école : créer l'agent ElevenLabs, coller son `agentId`, renseigner les critères avec les mêmes identifiants que l'analyse.

---

## 5. Règles de contenu
- Interface en **français**, tutoiement, phrases courtes et positives, comme sur les captures (« Transforme tes erreurs en points. »).
- **Aucun fait inventé** sur HEC, l'ESSEC ou l'ESCP : formats, durées, coefficients, classements. Si l'info n'est pas dans la config, affiche « À compléter ».
- Données utilisateur fictives, mais cohérentes avec les captures : Baptiste Puiferrat, 328/600, etc.

## 6. Garde-fous (non négociables)
- **Projet Lovable privé** ; pas de publication publique ni d'indexation. Si un déploiement est nécessaire pour la démo, demande-moi avant et ajoute `noindex`.
- **Mention discrète « Prototype »** : par exemple un petit badge dans la carte utilisateur ou en bas de la sidebar (« Prototype · B. Puiferrat »). Ce clone ne doit jamais pouvoir être confondu avec la vraie plateforme.
- **Ne récupère ni le code, ni les assets, ni les contenus** (vidéos, questions, images) de astprep.com. On reproduit l'apparence à partir des captures uniquement. Le logo est recréé en CSS ou SVG simple (pastille dégradée + triangle).
- **Pas de logos officiels** des écoles : uniquement des monogrammes texte.
- **Aucun secret** dans le code front ni dans le repo. Vérifie `.gitignore`.

## 7. Critères d'acceptation
- [ ] Une capture du Dashboard cloné posée à côté de `docs/screens/14-dashboard.png` est quasi indiscernable (même grille, mêmes rayons, mêmes couleurs, même typo).
- [ ] La sidebar contient exactement les entrées d'origine + « Jury IA · Nouveau ».
- [ ] `/jury-ia` affiche HEC, ESSEC et ESCP ; l'ESSEC est « Prêt », HEC et ESCP « Bientôt ».
- [ ] Un entretien vocal ESSEC démarre, la transcription s'affiche, puis « Terminer » mène au débrief.
- [ ] Le débrief affiche la vraie analyse ElevenLabs, ou le mode démo si elle est indisponible.
- [ ] Responsive correct jusqu'à 1280 px de large au minimum (la démo se fera sur un laptop). Le mobile n'est pas prioritaire.
- [ ] Projet privé, mention « Prototype » présente, aucun secret exposé.

---

## 8. Annexe : spécification page par page (à suivre en plus des captures)

> Toutes les interactions ci-dessous sont **en local** (état React + mock). Aucune ne doit appeler de backend, sauf Jury IA.

### 8.1 Comportements globaux (shell de l'app)
- **Sidebar repliable** : le bouton « ‹ » la réduit en colonne d'icônes, avec infobulles au survol, et un « › » permet de la rouvrir.
- **Sous-menu « Outils »** : dépliable en accordéon avec chevron. Les sous-items (Flashcards, Calcul mental, Lecture alphabétique) sont indentés, avec un filet vertical à gauche ; le sous-item actif est une carte blanche.
- **Sélecteur « TAGE MAGE »** : dropdown avec TAGE MAGE (actif) et TOEIC, marqué « Bientôt » et désactivé.
- **Recherche** : la barre ou `⌘K` / `Ctrl+K` ouvre une palette de commandes (shadcn `Command`) qui liste les pages et quelques cours ou sous-tests fictifs. Elle permet de naviguer.
- **Cloche** : popover « Notifications et nouveautés », avec 2 ou 3 entrées fictives, dont « Nouveau : Jury IA ».
- **Lune** : bascule le mode sombre. Fournis une version sombre cohérente : fond `#1A1430`, cartes `#2A2145` translucides, texte clair, violet inchangé. Mémorise le choix en `localStorage`.
- **Carte utilisateur** : le clic ouvre le menu de la capture 01, au-dessus de la carte. On y trouve l'avatar, le nom, l'email tronqué, puis les entrées Mon abonnement, Paramètres, Gérer mes cookies et Notifications et nouveautés, séparées par des filets. Chaque entrée ouvre un toast « Prototype ».
- **Titre de la topbar** : il reprend le nom de la page (« Dashboard », « Statistiques », « Générateur de session », « Cours du TAGE MAGE », « Parcours TAGE MAGE »…).
- **Transitions** : fondu et léger glissement à l'entrée des cartes, survol des cartes avec élévation d'ombre, survol des boutons avec léger éclaircissement.

### 8.2 Dashboard (capture 14)
- Titre « Bonjour, Baptiste. » ; sous-titre « Calibré sur environ 3 mois de préparation, tu as 10 tâches au programme aujourd'hui. »
- **Première rangée**, en 3 cartes :
  1. **Estimation TAGE MAGE** : 328 / 600, badge vert « ↗ +40 », courbe violette avec points (axe 250–400, dates 17 mai → 7 juin), infobulle au survol d'un point.
  2. **Série** : « Top 77 % · Record 4 j » avec une flamme, « 1 jour », mini-heatmap de points en haut à droite (le dernier en violet).
  3. **Carnet d'erreurs** : texte d'état vide, icône signet.
- **Deuxième rangée** :
  - **Programme du jour** (grande carte) : en-tête avec « ▷ Lancer ma session » et « Voir mon parcours → ». La liste défile ; chaque ligne a une case ronde à cocher, un titre, un type (Cours / Drill / Découverte), une durée et un chevron. **Cliquer sur la case coche la tâche**, avec une animation.
  - À droite : **Score par sous-test** (/ 60), avec 6 lignes (point de couleur, nom, barre, score : 35, 41, 34, 32, 21, 34), puis la carte sombre **« Ton Discord privé ASTPrep »** (badge « Réservé aux abonnés », ligne « ● En direct maintenant · Logique », bouton « Accéder au Discord privé »).
- **Ajout** : la bannière d'annonce Jury IA (voir section 3).

### 8.3 Statistiques (capture 06)
- Titre « Tes statistiques. » ; sous-titre « Tout ce que tu travailles devient un signal : … »
- **Rangée 1** :
  - Estimation 328/600 avec badge « +40 depuis ton premier blanc », courbe et lien « TAGE MAGE Blancs » ;
  - Jours d'activité « 12 jours », « Tu es 3349e », icône flamme ;
  - Temps total travaillé « -- » avec son texte d'état vide et une icône horloge.
- **Rangée 2** :
  - **Profil par sous-test** (/ 60) et note en bas « Moyenne de tes 5 derniers sujets… » ;
  - **Détail par thème** : onglets en pilule (Compréhension, Calcul, Raisonnement, Conditions min., Expression, Logique). Changer d'onglet change la liste de thèmes : nom tronqué avec « … », pourcentage, barre, « N question(s) ». Mets des valeurs variées, pas que des 0 %.
- **Rangée 3** : **Activité**, une heatmap façon GitHub sur 12 semaines (de « il y a 12 semaines » à « aujourd'hui »), avec une légende « Moins □□□□■ Plus » en 5 nuances de violet.

### 8.4 Sous-tests (capture 09)
- Titre « Travaille les sous-tests. » ; sous-titre « Les sous-tests sont composés de 15 questions et durent 20 minutes chacun. »
- **Toggle en pilule en haut à droite** : Généraliste (actif, violet) / Thématique.
- **Onglets des 6 sous-tests**, chacun avec son icône : Compréhension (livre), Calcul (#), Raisonnement (cerveau), Conditions min. (cible), Expression (crayon), Logique (étincelles).
- **Grille de 3 colonnes de cartes** (CDT 1, CDT 2…, préfixe selon le sous-test). Chaque carte a :
  - une pastille avec un point violet et un badge « Pas fait » ou « ● Fait » (vert) ;
  - un titre et « Compréhension de textes · 15 questions · 20 minutes » ;
  - **si la carte est faite** : « Mon score 16 / 60 | Moyenne ASTPrep 33 / 60 », les boutons « ✓ Voir la correction » (primaire) et « ↻ Refaire le sous-test » (secondaire), puis « ⏱ Sans chrono » ;
  - **sinon** : « ▷ Lancer le sous-test » et « ⏱ Sans chrono ».

### 8.5 TAGE MAGE Blanc (capture 08)
- Titre « Les TAGE MAGE Blancs, en conditions réelles. » ; sous-titre « 2 heures, 6 sous-tests, score sur 600… »
- **Filtre segmenté en haut à droite** : Tous / À faire (12) / Passés (10). Il **filtre vraiment la grille**.
- **Bannière sombre pleine largeur** : badge « ● Recommandé pour toi », « TAGE MAGE OCTOBRE 2026 » et « 2 heures », bouton « ⚡ Lancer ».
- **Grille de 4 colonnes**, cartes TAGE MAGE 1 à 22 :
  - **passés** : badge « ✓ Passé », « 90 questions, 6 sous-tests », séparateur, score violet (373, 320, 327, 353, 273, 313, 327, 287…), « Voir la correction » (primaire) et « ↻ Relancer » (lien) ;
  - **à faire** : bouton pleine largeur « Lancer → ».

### 8.6 Générateur (capture 07)
- Titre « Compose ta session sur mesure. »
- **Colonne gauche** :
  - **Nombre de questions** : gros chiffre « 15 questions (~20 min) » qui se met à jour (environ 1,33 min par question) et chips 10 / 15 / 20 / Personnalisé (champ numérique) ;
  - **Pool de questions** : grille 2×2 (Nouvelles, Déjà vues, Erreurs, Flaggées), chaque option avec icône, titre et description, plus une carte pleine largeur « Tout mélangé » avec badge « Recommandé ». Sélection unique, l'option active a un fond lavande et une bordure violette ;
  - **Mode** : Quiz / Entraînement libre.
- **Colonne centrale** :
  - **Sous-tests à inclure** : 5 lignes avec case à cocher, point de couleur, nom et compteur (Calcul 1550+, Raisonnement 250+, Conditions min. 850+, Expression 150+, Logique 600+). Sélection multiple ;
  - **Difficulté** : grille 2×2 (Facile, Moyen, Difficile, Mixte), avec description.
- **Colonne droite** :
  - **carte sombre « ● Ta session »** : gros chiffre « N question(s) », « Répartition par sous-test » qui **se calcule selon les cases cochées** (répartition équilibrée affichée en liste ou barres ; texte d'aide s'il n'y a aucune sélection), puis des chips récapitulatives « Difficulté : Moyen », « Pool : Tout mélangé », « Mode : Entraînement libre ». Le bouton « Lancer » est désactivé tant qu'aucun sous-test n'est coché ;
  - en dessous : carte « Relancer ma dernière session » avec son état vide.

### 8.7 Parcours (capture 12)
- Titre « Parcours TAGE MAGE » ; sous-titre « Calibré sur environ 3 mois de préparation… »
- **En haut à droite** : segmenté Aujourd'hui / Semaine / Mois, et bouton contour « ↻ Régénérer ».
- **Bandeau** : « ● Ton parcours personnalisé », « Jour 1 / 91 », badge vert « À jour », et « Objectif 400 / 600 » à droite.
- **Carte principale** :
  - en-tête « Session du jour : 04/11/2026 » et « 10 tâches · 81 min » ;
  - liste défilante dont chaque ligne a une icône selon le type (✦ découverte, ▷ cours, ⚡ drill), une méta grise (« Cours · 11 min »), un titre et un chevron ;
  - bouton pleine largeur « Lancer ma session du jour → ».
- **Carte droite « À rattraper »** : horloge et badge vert « ● À jour », texte « Ton parcours est à jour. »
- **Vues Semaine et Mois** : grille simple de jours avec pastilles de tâches. Pas besoin d'être fidèle, ces vues ne sont pas sur les captures.

### 8.8 Cours (capture 11)
- Titre dynamique « Compréhension de textes » et « 6 leçons » en gris ; sous-ligne « Choisis ton sous-test pour voir les leçons associées. »
- **Onglets avec compteurs** : Compréhension 6, Calcul 51, Raisonnement 24, Conditions min. 14, Expression 9, Logique 9. L'onglet change le titre et la liste.
- **Carte accordéon « Cours généraux »** : pastille, « 6 leçons · 0 terminée », barre de progression et chevron. Elle contient des lignes numérotées 01 à 06, avec bouton ▷, titre, durée et chevron.
- **Carte droite « Ton avancement »** : trophée, « 0 / 6 leçons », nom du sous-test, barre.

### 8.9 Cours en direct (capture 10)
- Titre « Cours en direct » ; sous-titre « Des sessions en ligne animées par les coachs ASTPrep… »
- **Colonne gauche « À venir »** (« 5 sessions »), des cartes avec :
  - un titre ;
  - un badge d'état (« ● En direct » rouge, ou « Bientôt » lavande) ;
  - un badge de catégorie coloré (Logique, Conditions min., Raisonnement…) ;
  - une description ;
  - la date (icône calendrier), un avatar rond avec le prénom et l'initiale du coach (**prénoms fictifs**) ;
  - le bouton « ↗ Rejoindre maintenant » si la session est en direct, « Rejoindre le live » sinon.
- **Colonne droite « Replays »** (« 92 replays ») :
  - chips filtres : Tous 92, Calcul 38, Raisonnement 10, Conditions min. 32, Logique 9, Général 3. Elles **filtrent la liste** ;
  - liste défilante de cartes replay : pastille ▷, titre tronqué, badge de catégorie, « date · avatar coach », description tronquée, chevron.

### 8.10 Parrainage (capture 13)
- Titre centré « Baptiste, parraine tes amis et fais baisser ton abonnement. » ; sous-titre « Ton ami reçoit une réduction, et tu reçois exactement la même. »
- **Grande carte à bordure violette** : code géant espacé en violet (code fictif, **pas** celui de la capture), bouton « Copier mon code » (copie dans le presse-papier + toast « Code copié »), lien de parrainage fictif et « Copier le lien ».
- **Rangée de 3 boutons contour** : Partager, WhatsApp, Mail.
- **Carte « Ce que tu gagnes »** : 3 lignes (Mensuel → 10 €, Pack 3 mois → 20 €, Saison → 50 €) et note de bas de carte.
- **Carte droite** :
  - bloc « Les Oraux offerts » (icône cadeau, texte, barre, « 0 sur 5 amis abonnés », « Plus que 5 ») ;
  - section « Tes amis parrainés » avec « 0 € de crédit » et son état vide.

### 8.11 Carnet d'erreurs (capture 05)
- Titre « Transforme tes erreurs en points. » ; sous-titre de la capture.
- **État vide centré** : grande carte avec pastille signet, « Ton carnet d'erreurs est vide. » et le texte explicatif.
- Prévois aussi une variante remplie, accessible avec `?demo=1` : liste d'erreurs fictives par sous-test et bouton « Composer une session d'erreurs ».

### 8.12 Outils : Flashcards (capture 04)
- Titre « Révise tes flashcards, sous-test par sous-test. »
- **Onglets** : « # Calcul 250 » et « ✎ Expression 378 ».
- **Grille de 3 colonnes de decks** : pastille avec un point bleu (`#6F8FE8`), badge « Pas fait », titre, « N cartes · ~X min », bouton pleine largeur « ▷ Lancer le deck ».
- Le bouton ouvre une **modale de flashcard** qui se retourne au clic, avec 3 ou 4 cartes fictives et une navigation Suivant / Précédent.

### 8.13 Outils : Calcul mental (capture 03)
- Titre « Aiguise tes réflexes de calcul. » ; bouton « ▷ Lancer la session » en haut à droite.
- **Carte Opération** : grille 2×2 (Addition +, Soustraction −, Multiplication ×, Mix des 3 avec pastille violette pleine et ✓). Sélection unique.
- **Cartes Difficulté** (Facile / Moyen / Difficile) et **Durée** (2 / 5 / 10 min), en chips.
- **Panneau sombre** :
  - « Tes meilleurs scores » avec son état vide ;
  - « Classement » : 5 lignes de rang, avatar à initiales, nom fictif et score.
- **Bonus** : « Lancer la session » démarre un vrai mini-jeu (calcul affiché, champ réponse, chrono, score). C'est facile à coder et ça renforce l'effet « vraie plateforme ».

### 8.14 Outils : Lecture alphabétique (capture 02)
- Titre « Trouve le rang de chaque lettre, le plus vite possible. »
- **Carte Durée** (chips).
- **Carte « Pourquoi ça compte »** : grille A–Z de 13 × 2 tuiles, chaque tuile avec la lettre et son rang.
- **Panneau sombre** : meilleurs scores et classement.
- **Bonus** : même principe de mini-jeu (une lettre s'affiche, on tape son rang).

### 8.15 Jury IA
Voir la section 4. Elle s'appuie sur les mêmes composants : chips, cartes de sélection 2×2 façon Calcul mental, panneau sombre façon « Tes meilleurs scores », grands chiffres façon Statistiques.

---

**Commence par lire le brief et les 14 captures, puis propose-moi le plan (section 0). Ne lance rien dans Lovable avant mon OK.**

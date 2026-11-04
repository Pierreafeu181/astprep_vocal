# Brief projet : Simulateur d'entretien oral AST par école (PoC ASTPrep)

> Brief de travail pour Claude Code. Il contient le contexte, ce qui est déjà construit, ce qu'il reste à faire et les décisions déjà prises.
> Rédigé le 4 nov. 2026, après un entretien d'alternance chez ASTPrep. Le CEO m'a donné accès à la plateforme et m'a proposé soit d'améliorer une fonctionnalité existante, soit d'en créer une nouvelle en vibe code. **J'ai choisi de créer une nouvelle fonctionnalité.**

---

## 1. Contexte en 30 secondes

- **ASTPrep** (astprep.com) est une plateforme de préparation au TAGE MAGE et aux admissions parallèles (AST) en école de commerce, lancée en septembre 2024.
  - Fondateurs : Simon Lamouche (« Simon AST ») et Grégoire Le Roch.
  - Environ 5 000 élèves revendiqués.
  - Prix : 97 €/mois, 249 € les 3 mois, 549 € l'année.
- **Ce qu'ils ont déjà** :
  - 6 000 questions corrigées en vidéo et 21 TAGE MAGE blancs ;
  - carnet d'erreurs, statistiques, parcours, lives (10 h par semaine) ;
  - assistant IA entraîné sur leurs cours ;
  - TOEIC ;
  - dossier et oraux, avec **2 simulations d'oral humaines, uniquement dans l'offre annuelle** ;
  - un Discord public d'environ 14 000 membres ;
  - un simulateur d'admissibilité gratuit (simulateur.astprep.com).
- **La fonctionnalité** : un **jury vocal IA spécifique à chaque école**. L'élève passe un entretien à l'oral, avec des relances, puis reçoit un **débrief noté** avec des réponses réécrites à partir de son propre parcours.

## 2. Pourquoi cette fonctionnalité (résultats de la recherche marché)

- **La prépa au TAGE MAGE seule est une commodité.**
  - AST+ et Teachero sont des plateformes complètes et **gratuites** (Teachero : environ 8 300 questions et 25 blancs).
  - Les abonnements payants vont de 15 à 57 € par mois.
  - La différenciation ne passe donc plus par la banque de questions.
- **Les oraux sont un besoin mal servi** :
  - c'est souvent le premier vrai entretien de l'élève, avec beaucoup de stress ;
  - le « pourquoi cette école ? » est généralement mal préparé ;
  - l'offre est quasi binaire : 2 simulations humaines chez ASTPrep (en annuel), ou une prépa haut de gamme à 1 600 € et plus (Ipesup, Laurea, Hello Prépa).
  - Il n'existe rien pour **s'entraîner dix fois** avant le jour J.
- **L'IA générative est encore peu présente dans les prépas françaises.** Les meilleurs acteurs du GMAT (Magoosh, Target Test Prep, e-GMAT) ont des tuteurs IA, mais personne ne propose de simulateur d'oral vocal.
- **Business** :
  - upsell naturel, au moment où l'élève est le plus prêt à payer (après l'admissibilité) ;
  - **complète** les simulations humaines au lieu de les remplacer : l'IA pour les répétitions, les profs pour les simulations finales, avec le débrief IA transmis aux profs.
- **Une autre idée sur la table, pour plus tard** : un « Copilote de candidature » qui relierait le score live à des chances par école et à une timeline des concours. Attention, il recoupe partiellement leur simulateur gratuit actuel ; il faut d'abord vérifier ce que ce dernier fait.

## 3. Ce qui est DÉJÀ construit (ElevenLabs Conversational AI)

Deux agents vocaux existent sur mon compte ElevenLabs :

| Agent | agent_id | Contenu |
|---|---|---|
| ASTPrep PoC – Jury ESSEC (AST) | `agent_4001m6k72ymffqnscw4vykmqxj30` | Entretien de motivation condensé à environ 15 min (le vrai dure environ 45 min) |
| ASTPrep PoC – Jury EDHEC (AST) | `agent_0601m6k73aprfeea645x5bx7x9bm` | Présentation de 4 min avec mot imposé, puis entretien individuel |

Liens de test :
- ESSEC : https://elevenlabs.io/app/talk-to?agent_id=agent_4001m6k72ymffqnscw4vykmqxj30
- EDHEC : https://elevenlabs.io/app/talk-to?agent_id=agent_0601m6k73aprfeea645x5bx7x9bm

Configuration actuelle des deux agents :
- **Langue** : `fr`. Le LLM de conversation est celui par défaut d'ElevenLabs, choisi pour sa faible latence.
- **Voix : provisoire.** Ce sont des voix standard multilingues (Eric pour l'ESSEC, River pour l'EDHEC), avec un léger accent anglophone. À remplacer par des voix françaises de la bibliothèque, après les avoir ajoutées au compte (par exemple « Julien - Service client professionnel » ou « Jade - Customer Service »).
- **Analyse post-appel** (LLM d'analyse `claude-sonnet-5-5`, résumé en français) :
  - **5 critères d'évaluation** notés de 0 à 4 (`numeric_uniform`, `max_score` 4) ;
  - **data collection** : `note_sur_20` (number), `debrief_points_forts` (string), `debrief_axes_amelioration` (string).
- Le **prompt système** contient la fiche école directement (pas de RAG, pour limiter la latence).

### Grille de débrief

**ESSEC** : chaque critère est noté sur 4, ce qui donne une note sur 20.
1. `structure_clarte` : structure et clarté (méthode STAR, longueur adaptée)
2. `coherence_projet` : cohérence du projet professionnel avec le parcours
3. `connaissance_ecole` : choix de l'école et de la filière justifié de façon spécifique, pas générique
4. `aisance_orale` : fluidité, tics de langage, réponses récitées
5. `questions_difficiles` : gestion des relances, de l'échec, du plan B

**EDHEC** : même principe.
1. `presentation_mot` : intégration naturelle du mot imposé dans la présentation
2. `coherence_projet`
3. `connaissance_ecole`
4. `aisance_orale` : aisance orale et adaptabilité
5. `travail_groupe` : posture dans un exemple de décision collective

Règle clé du débrief : les meilleures réponses sont réécrites **uniquement à partir du vrai parcours du candidat**. On n'invente jamais d'expérience.

## 4. Fiches école (contenu des prompts)

> ⚠️ Les sources sont principalement Thotis, qui vend aussi de la préparation. **À faire valider par un prof ASTPrep avant la démo.** Règle absolue dans les prompts : l'IA n'invente aucun fait sur l'école (cours, classements, chiffres, professeurs).

**ESSEC, admission sur titre (MiM / PGE)**
- Entretien de motivation et de personnalité d'environ 45 min, coefficient 2, sur le campus de Cergy. Frais de concours : 180 €.
- Jury d'au moins 2 personnes : enseignant, membre de l'administration, diplômé ou professionnel, parfois un élève. Le jury a lu le dossier, le CV et l'Essai.
- Déroulé : présentation de 2 à 3 min, parcours, projet professionnel, choix de filière, actualité et mise en perspective.
- Filières :
  - **Flexible** : 2 à 3 ans ;
  - **Intensive** : 1 an, accessible avec au moins 6 mois d'expérience professionnelle et au moins 10 semaines d'expérience académique à l'international.
- Ce que le jury évalue : solidité du raisonnement, cohérence entre le passé et l'ambition, maturité des choix, compréhension du MiM.
- Campus : Cergy, La Défense, Rabat, Singapour. Le diplôme est le même que pour les élèves issus de prépa.
- Questions types :
  - « Pourquoi l'ESSEC et pas une autre ? »
  - « Pourquoi cette filière ? »
  - « Pourquoi le management ? »
  - « Quel est ton projet précis ? »
  - « Que feras-tu si tu n'es pas admis ? »
  - « Raconte un échec. »

**EDHEC, AST (entrée en M1 du PGE)**
- La « Trilogie », coefficient 15, en trois temps :
  1. présentation de 4 min avec un mot tiré au hasard ;
  2. exercice collectif de décision d'environ 40 min (**non simulé dans le PoC**) ;
  3. entretien individuel d'environ 20 min.
- Oral d'anglais, coefficient 10. LV2 en option, coefficient 2. Frais : 180 €.
- Ce que le jury évalue : structure du discours, intégration de l'imprévu (le mot), cohérence du parcours et du projet, choix de l'EDHEC. Les réponses récitées sont pénalisées.
- Campus : Lille et Nice, avec des sites à Paris et Londres. Spécialisations : finance, marketing, stratégie, conseil, data.

## 5. Objectif du PoC (ce que je dois livrer)

Une **web app de démo** qui enveloppe les agents dans l'univers ASTPrep, pour la montrer au CEO. Elle doit prouver trois choses :
1. un entretien vocal avec une IA est **assez naturel** pour s'entraîner, avec une latence inférieure à environ 2 s ;
2. les questions sont **spécifiques à l'école** et rebondissent sur le parcours de l'élève ;
3. le **débrief** est assez bon pour qu'un prof ASTPrep le valide.

### Parcours utilisateur
1. **Choix** : l'élève choisit l'école (ESSEC ou EDHEC) et le niveau du jury (bienveillant, neutre ou déstabilisant ; optionnel en v1).
2. **CV** (v1.1) : l'élève colle le texte de son CV, qui est injecté dans l'agent via les *dynamic variables* ou les *overrides* du prompt.
3. **Consentement** : bandeau indiquant que l'entretien est enregistré et transcrit pour produire le débrief. L'audio n'est pas conservé côté app.
4. **Entretien** : interface vocale avec un indicateur « le jury parle / vous parlez », un chrono et un bouton pour terminer.
5. **Débrief** : note sur 20, radar des 5 critères, points forts avec citations, 3 moments clés avec une réponse réécrite, et transcription repliable.
6. **Historique** (bonus) : la progression d'une simulation à l'autre.

### Stack proposée
- **Next.js** (App Router) + TypeScript + Tailwind.
- **SDK ElevenLabs React** (`@elevenlabs/react`, hook `useConversation`) pour la conversation dans le navigateur. Vérifier la doc à jour.
- **Côté serveur** (route API Next) :
  - générer une **signed URL**, si l'agent est passé en privé, pour ne pas exposer la clé API ;
  - récupérer la conversation terminée et son analyse via l'API ElevenLabs (endpoint conversations, avec le `conversation_id` renvoyé par le SDK) ;
  - **attendre la fin de l'analyse** : faire du polling tant que le statut n'est pas `done`.
- **Variables d'environnement** : `ELEVENLABS_API_KEY`, `NEXT_PUBLIC_AGENT_ID_ESSEC`, `NEXT_PUBLIC_AGENT_ID_EDHEC`.
- Pas de base de données pour la démo. Si l'historique est implémenté, utiliser Supabase plus tard.

### Charte visuelle (d'après une capture de la plateforme)
- Fond clair lavande et rose pastel en dégradé.
- Cartes blanches translucides de type glassmorphism, coins très arrondis, ombres douces.
- Accent **violet** pour les courbes et les chiffres clés ; badge vert pour le progrès (« +40 depuis ton premier blanc »).
- Typographie sans-serif moderne (de type Poppins ou Inter), avec de grands titres comme « Tes statistiques. ».
- Barre latérale avec les sections Apprendre, S'entraîner et Progresser. La fonctionnalité irait naturellement dans **Progresser** ou dans un nouvel onglet « Oraux ».
- Ton : tutoiement dans l'interface, alors que le jury vouvoie pendant l'entretien.
- Couleurs exactes à relever sur la plateforme avec les outils de développement du navigateur ; ne pas les inventer.

## 6. Plan de travail

1. [ ] Tester les 2 agents moi-même, en entretien complet, et noter la latence, la pertinence des relances et la qualité du débrief.
2. [ ] Remplacer les voix par des voix françaises.
3. [ ] Faire relire les fiches école par un prof ASTPrep, ou au minimum les recouper avec les sites officiels de l'ESSEC et de l'EDHEC.
4. [ ] Initialiser le projet Next.js : page de choix, page d'entretien (SDK), page de débrief.
5. [ ] Route API pour la signed URL, puis route API pour récupérer l'analyse, avec polling.
6. [ ] Écran de débrief : note, radar, points forts, moments clés, transcription.
7. [ ] v1.1 : injection du CV dans le prompt via les dynamic variables.
8. [ ] Validation avec 3 ou 4 personnes qui préparent les AST, plus un prof ASTPrep qui relit 2 ou 3 débriefs.
9. [ ] Mesurer le **coût réel par entretien** (minutes ElevenLabs plus LLM) pour l'argument business.
10. [ ] Préparer la démo pour le CEO (voir la section 8).

## 7. Risques et garde-fous

- **Données personnelles et RGPD** :
  - consentement explicite avant l'entretien ;
  - pas de stockage audio côté app ;
  - vérifier les paramètres de rétention ElevenLabs ;
  - hébergement européen si possible en production.
- **Hallucinations sur les écoles** : les faits sont limités à la fiche dans le prompt. Si une question sort du cadre, le jury ne doit pas inventer. Une seule erreur factuelle en démo suffit à ruiner la crédibilité.
- **Débrief générique** : c'est le risque principal pour le produit. Il faut calibrer la grille avec les profs d'ASTPrep, qui font déjà les simulations humaines.
- **Latence** : au-delà d'environ 2 s de délai, l'échange ne ressemble plus à un entretien. Garder un LLM rapide pour la conversation et réserver le modèle lourd au débrief.
- **Périmètre** : pas de caméra, pas de comptes utilisateurs et pas plus de 2 écoles en v1.

## 8. Démo devant le CEO (5 min)

1. **30 s : le problème.** Les oraux sont stressants et mal préparés. Il n'y a que 2 simulations humaines, uniquement dans l'offre annuelle. Si possible, citer le nombre de questions sur les oraux posées sur le Discord (à collecter avec la recherche Discord sur « oral », « entretien », « jury », « pourquoi l'école », en anonymisant).
2. **3 min : démo en direct.** Je passe un entretien « ESSEC » avec mon vrai parcours (je suis moi-même à l'ESSEC).
3. **1 min : le débrief** s'affiche, avec la réécriture d'une de mes réponses.
4. **30 s : le business.** Coût par entretien, upsell dans les offres Intensif et Annuel, et le fait que ça complète les profs sans les remplacer. Possibilité d'intégrer la fonctionnalité dans la timeline du futur Copilote de candidature (oraux débloqués quand les dates approchent).

## 9. Questions ouvertes à poser au CEO

- Combien d'élèves passent des oraux chaque année, et quelles écoles reviennent le plus ?
- Les profs accepteraient-ils de relire les débriefs IA ou de calibrer la grille ?
- Quelle stack utilise la plateforme (pour une intégration future) ?
- Que fait exactement leur onglet « Générateur » ? Et leur assistant IA ?
- Règles du TAGE MAGE en 2026 : 2 ou 4 passages par an, avec ou sans pénalités ? Les sources se contredisent.

## 10. Sources

- ASTPrep : https://astprep.com · avis : https://fr.trustpilot.com/review/astprep.com · simulateur : https://simulateur.astprep.com
- Oraux EDHEC : https://thotismedia.com/edhec-business-school-tout-savoir-sur-les-oraux-ast/
- Oraux ESSEC : https://thotismedia.com/essec-business-school-tout-savoir-sur-les-oraux-ast/
- Comparatif des prépas en ligne (nov. 2026) : https://www.planetegrandesecoles.com/preparation-tage-mage-en-ligne-meilleures-plateformes
- Teachero (gratuit) : https://www.teachero.fr · Magoosh GMAT : https://gmat.magoosh.com · Target Test Prep : https://gmat.targettestprep.com
- FAQ TAGE MAGE (FNEGE) : https://www.tagemage.fr/page/foire-aux-questions

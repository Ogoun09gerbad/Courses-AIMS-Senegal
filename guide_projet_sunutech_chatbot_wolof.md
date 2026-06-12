# Guide de projet — Team SunuTech : Chatbot wolof (Local Language Chatbot)

**Projet :** Real LLM Deployment and Standard Industrial Methodologies (suite du Lab Session 7)
**Cas d'usage retenu :** Chatbot en wolof pour les sujets du quotidien (culture, éducation, agriculture, santé, transport) + correction orthographique
**Modèle de base :** Qwen/Qwen3-0.6B + LoRA

---

## 0. Vue d'ensemble du pipeline

L'ordre logique du projet est le suivant. Chaque étape dépend de la précédente — c'est pour ça qu'on suit ce séquencement.

```
1. Données (Binta)          -> data/wolof_*.jsonl, chat_*.json, splits/, config YAML
2. Entraînement LoRA (Ger)   -> outputs/qwen3_0_6b_wolof_lora/best_adapter
3. Context state machine     -> déjà fait (fichier fourni), à intégrer au Space
4. Évaluation (Danielle)     -> métriques + analyse par catégorie
5. Push Hugging Face (Marie Paul) -> modèle + model card
6. Space (Maimouna)          -> app connectée au modèle du Hub
7. Rapport + notes individuelles -> tout le monde
```

Les étapes 3 et 6 (Maimouna) peuvent démarrer **en parallèle** de l'étape 2, car elles dépendent surtout des données, pas encore du modèle entraîné.

---

## 1. Cas d'usage (rappel pour le rapport)

**Nom :** Assistant conversationnel en wolof pour les sujets du quotidien
**Utilisateurs :** Locuteurs wolof, contexte sénégalais
**Domaines couverts :** culture, éducation, agriculture, santé, transport (catégories du dataset synthétique fourni), + compétence générale en wolof (aya), + correction orthographique (soynade)
**Pourquoi un petit modèle suffit :** périmètre limité (Q/R courtes, sujets fermés), compute limité (CPU sur Space gratuit)

---

## 2. Binta — Données (`src/download_datasets.py`)

**Objectif :** produire les 3 sources séparées, les fichiers chat, les splits et le YAML de config.

**À faire (sur Kaggle, car l'accès à Hugging Face Hub demande internet) :**

```bash
python src/download_datasets.py all \
  --max-aya-examples 500 \
  --max-soynade-examples 500 \
  --validation-ratio 0.05 \
  --eval-ratio 0.05
```

**Sources et leur rôle :**

| Source | Fichier | Rôle | Origine |
|---|---|---|---|
| `aya` | `data/wolof_aya.jsonl` | Source 1 — générale | CohereLabs/aya_dataset (filtré wolof) |
| `synth` | `data/wolof_synth.jsonl` | Source 2 — synthétique | `data/syntetic_wolof_instruct_data.jsonl` (déjà fourni, 5300 lignes) |
| `soynade` | `data/wolof_soynade.jsonl` | Source 3 — domaine (correction orthographique) | soynade-research/Wolof-Non-Standard-Orthography |

**Vérifications à faire après l'exécution :**

- Les fichiers `data/chat_aya.json`, `data/chat_soynade.json`, `data/chat_synth.json` existent et ont le format `messages: [system, user, assistant]`.
- `data/splits/chat_<source>_{train,validation,eval}.json` existent pour les 3 sources.
- `data/splits/eval_all.jsonl` et `configs/wolof_training_config.yml` existent.
- Vérifier 10-15 lignes de `wolof_aya.jsonl` à la main : est-ce bien du wolof (pas du français/anglais mal filtré) ?
- Noter le nombre de lignes train/val/eval par source — `aya` et `soynade` seront probablement petits (peu d'exemples eval, à signaler comme limite dans le rapport).

**Livrable :** remplir `data/README.md` avec un tableau (source / nombre d'exemples / licence / méthode de nettoyage / rôle). Ce tableau sera réutilisé tel quel dans le rapport et la model card.

---

## 3. Ger — Entraînement LoRA + coordination + context state machine

**Objectif 1 : entraînement**

Une fois que les fichiers de Binta existent, test rapide d'abord :

```bash
python train_lora_assistant_only.py \
  --config configs/wolof_training_config.yml \
  --model-choice qwen \
  --max-train-examples 100 \
  --epochs 1 \
  --no-checkpoint-eval \
  --no-use-config-hparams
```

Puis l'entraînement complet :

```bash
python train_lora_assistant_only.py \
  --config configs/wolof_training_config.yml \
  --model-choice qwen
```

**Points à vérifier :**

- Dans `configs/wolof_training_config.yml`, section `training_data` : comme `synth` (catégorie "culture" = 2100/5300) est beaucoup plus gros que `aya`/`soynade`, vérifier que les poids ne laissent pas une seule source dominer l'entraînement.
- Masquage assistant-only : écrire un petit test qui charge un batch et vérifie que les labels des tokens `system`/`user`/`padding` valent bien `-100` (10% de la note).
- Suivi avec TensorBoard : `tensorboard --logdir outputs/qwen3_0_6b_wolof_lora/runs --port 6006`.

**Objectif 2 : context state machine**

Le fichier redessiné (`RetrievalContextStateMachine`) est déjà prêt — il faut :

1. Le placer dans `src/context_state_machine.py` (remplace l'ancienne version).
2. Construire l'index au démarrage du Space avec `load_corpus("data/splits")` (train+validation seulement, jamais eval).
3. Transmettre le module à Maimouna pour l'intégration dans `hf_space/app.py` (section 6).

**Objectif 3 : coordination**

- Assembler le rapport final (`templates/PROJECT_REPORT_TEMPLATE.md`) à partir des contributions de chacun.
- Vérifier que chaque note individuelle répond bien aux 5 questions (section 8).
- Suivre les dépendances : relancer Danielle dès que `best_adapter` existe, relancer Marie Paul dès que l'éval est validée, etc.

---

## 4. Danielle — Évaluation (`evaluation.py`)

**À faire une fois `best_adapter` disponible :**

```bash
python evaluation.py \
  --data data/splits/eval_all.jsonl \
  --generate \
  --model-choice qwen \
  --adapter auto
```

**Analyses attendues :**

- Métriques globales : Exact Match, token F1, BLEU, ROUGE-L.
- **Ventilation par catégorie** (culture, éducation, agriculture, santé, transport, langue, orthographe) : quelles catégories sont faibles ? C'est le point central pour la section "limitations" du rapport.
- 3-5 exemples qualitatifs par catégorie, style conversationnel (c'est un chatbot du quotidien, pas un examen académique).
- Avec le `context_state_machine.py`, calculer le **taux d'abstention** sur le split eval : combien de questions tombent sous `LOW_CONFIDENCE_THRESHOLD` (confiance < 0.12) ? Un chiffre simple à inclure dans le rapport.

**Livrable :** tableau de métriques (global + par catégorie) + section "exemples qualitatifs" + section "cas d'échec".

---

## 5. Marie Paul — Hugging Face Hub + Model Card

**À faire une fois l'adaptateur validé par Danielle :**

```bash
huggingface-cli login
python push_to_hub.py --check-auth
python push_to_hub.py \
  --repo-id YOUR_USERNAME/YOUR_MODEL_NAME \
  --model-choice qwen \
  --adapter-dir auto \
  --checkpoint best \
  --public
```

**Remplir `templates/MODEL_CARD_TEMPLATE.md` :**

- **Target task :** chatbot conversationnel en wolof, sujets du quotidien + correction orthographique.
- **Target users :** locuteurs wolof, contexte sénégalais.
- **Out-of-scope use :** pas de conseil médical/diagnostic (cohérent avec le disclaimer santé du `context_state_machine.py`), pas de sujets hors des 5 catégories couvertes.
- **Data Methodology / Data Splits :** copier le tableau de Binta.
- **Training Configuration :** copier les hyperparamètres réels de `configs/wolof_training_config.yml` (LoRA rank, alpha, learning rate, epochs, etc.).
- **Evaluation :** copier les métriques de Danielle.
- **Deployment :** liens du repo HF model + Space (une fois prêts par Maimouna).

---

## 6. Maimouna — Space (`hf_space/app.py`)

**Objectif :** adapter le Space Gradio fourni pour le chatbot wolof, en intégrant le `context_state_machine.py`.

**À faire :**

1. Démarrage : charger le corpus de récupération.
   ```python
   from src.context_state_machine import load_corpus, RetrievalContextStateMachine
   corpus_rows = load_corpus("data/splits")
   STATE_MACHINE = RetrievalContextStateMachine(corpus_rows)
   ```

2. Ajouter un sélecteur de catégorie basé sur `STATE_MACHINE.available_categories()`.

3. Dans la fonction de génération, appeler `assess()` avant de construire le prompt :
   ```python
   assessment = STATE_MACHINE.assess(category, user_message)
   if assessment.abstain_reason == "safety":
       return assessment.disclaimer  # refus direct, pas de génération
   augmented = STATE_MACHINE.augment_instruction(assessment)
   prompt = render_prompt(augmented)
   ```

4. Afficher dans l'interface (zone dépliable ou texte secondaire) :
   - les exemples récupérés (instruction / réponse / source / confiance),
   - le score de confiance global,
   - le disclaimer santé si présent.

5. Remplacer `YOUR_USERNAME/YOUR_LORA_MODEL_REPO` par le repo de Marie Paul une fois publié.

**Pour la démo finale, préparer un scénario couvrant :**

- une question par grande catégorie (culture, éducation, agriculture, santé, transport),
- un exemple de correction orthographique,
- un exemple de refus de sécurité (catégorie santé, question type "dosage").

---

## 7. Méthodologie obligatoire (rappel rapide)

1. **3 sources séparées et documentées** (général / synthétique / domaine) — Binta.
2. **Format chat explicite** system/user/assistant — généré automatiquement par `download_datasets.py`.
3. **Splits reproductibles**, seed fixe, par famille de source (déjà géré par le script).
4. **Masquage assistant-only** : labels `-100` sur system/user/padding — Ger vérifie.
5. **Évaluation** : EM, F1, BLEU, ROUGE-L + exemples qualitatifs + limites — Danielle.
6. **Déploiement** : adaptateur sur le Hub + Space connecté au Hub (pas un checkpoint local) — Marie Paul + Maimouna.

---

## 8. Notes individuelles — rappel

Chaque personne écrit une demi-page (max 1 page) répondant à :

1. Quelle partie exacte du projet as-tu implémentée ou améliorée ? (fichiers, fonctions, commits)
2. Quel choix technique as-tu fait, et quelle alternative as-tu rejetée ?
3. Quel problème ou échec as-tu observé, et comment l'as-tu diagnostiqué ?
4. Comment as-tu vérifié que ta contribution fonctionne ? (une métrique, un test, une capture d'écran, une sortie de commande, un exemple qualitatif)
5. Si tu avais un jour de plus, qu'améliorerais-tu en priorité ?

L'instructeur peut poser une question individuelle pendant la soutenance basée sur cette note — chacun doit pouvoir expliquer précisément sa partie.

---

## 9. Checklist finale avant soumission

- [ ] `data/README.md` rempli (3 sources documentées)
- [ ] `configs/wolof_training_config.yml` existe et correspond aux données réelles
- [ ] `data/splits/eval_all.jsonl` existe
- [ ] `outputs/.../best_adapter` existe localement
- [ ] Test du masquage `-100` fait et documenté
- [ ] Métriques d'évaluation (globales + par catégorie) calculées
- [ ] Repo modèle Hugging Face public (ou accessible à l'instructeur)
- [ ] Space Hugging Face fonctionnel, connecté au repo du Hub
- [ ] `context_state_machine.py` intégré au Space (exemples récupérés + confiance + disclaimer visibles)
- [ ] Model card complète
- [ ] Rapport complet (cas d'usage, données, entraînement, évaluation, déploiement, limites, améliorations)
- [ ] 5 notes individuelles écrites

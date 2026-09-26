# 🏨 Hotel RAG Assistant

Assistant virtuel pour l'**Hôtel Le Belvédère** (bord du lac d'Annecy), construit avec une architecture **RAG (Retrieval-Augmented Generation)** : au lieu de laisser un LLM inventer des réponses, on lui fournit uniquement les extraits de documentation pertinents avant qu'il ne réponde.

## 📌 Contexte du projet

Le propriétaire de l'hôtel dispose de 5 fichiers PDF de documentation (horaires, tarifs, spa, wifi, etc.) rédigés pour être lus par des humains. L'objectif : construire un assistant capable de répondre aux questions des clients **en s'appuyant uniquement sur cette documentation**, sans halluciner.

Le projet compare trois approches :

1. **LLM seul** — sans aucun contexte → réponses génériques ou inventées
2. **Tout le contexte dans le prompt** — toute la documentation envoyée à chaque question → réponses correctes mais lentes et peu scalables
3. **RAG** — recherche sémantique pour ne récupérer que les sections pertinentes → réponses précises, rapides, et honnêtes ("je ne sais pas" quand l'info n'existe pas)

## 🛠️ Stack technique

- **Hugging Face `transformers`** — LLM `Qwen/Qwen2.5-0.5B-Instruct` pour la génération de texte
- **`sentence-transformers`** — modèle `paraphrase-multilingual-MiniLM-L12-v2` pour les embeddings sémantiques (384 dimensions)
- **`pypdf`** — extraction de texte depuis les documents PDF
- **`pandas`** — structuration des rubriques extraites
- **`scikit-learn`** (t-SNE) — visualisation des embeddings en 2D
- **`matplotlib`** — graphiques

## ⚙️ Fonctionnement du pipeline RAG

1. **Extraction** : chaque page des 5 PDF est convertie en texte et nettoyée (suppression des titres/pieds de page redondants) → 15 rubriques structurées
2. **Mise en forme Markdown** : chaque rubrique devient une section `## titre` + texte
3. **Embeddings** : les 15 rubriques sont converties en vecteurs sémantiques (384 dimensions)
4. **Recherche** : à chaque question, calcul de similarité (produit scalaire) entre la question et les rubriques → récupération des `top_k` plus pertinentes
5. **Génération** : les rubriques récupérées sont injectées dans le prompt final envoyé au LLM, avec une consigne stricte interdisant d'inventer une réponse hors documentation

## 🚀 Installation

```bash
uv add transformers torch sentence-transformers pypdf ipywidgets matplotlib scikit-learn pandas
```

## ▶️ Utilisation

Ouvrir le notebook `hotel-rag-assistant.ipynb` et exécuter les cellules dans l'ordre. Les PDF de documentation doivent se trouver dans le dossier `data/`.

```python
answer, sources = answer_question("Le wifi est-il gratuit ?")
```

## 📊 Résultats

| Approche | Contexte envoyé | Précision | Vitesse |
|---|---|---|---|
| LLM seul | Aucun | ❌ Invente | Rapide |
| Prompt complet | ~1400 mots (tout) | ✅ Correct | Lent |
| RAG | ~150-200 mots (top-k) | ✅ Correct | Rapide |

## 📁 Structure

```
.
├── projet_04.ipynb      # Notebook principal
├── data/                 # 5 PDF de documentation de l'hôtel
├── utils.py              # Fonctions utilitaires (print_chat, etc.)
└── image.png
```

## 🎓 Origine

Projet réalisé dans le cadre du "cahier de vacances" Machine Learnia (Projet 04 - RAG & LLM).

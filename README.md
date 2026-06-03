# MA-Rank: Multi-Agent Resume Screening and Skill Mining System

**Arizona State University**

| Name | Email |
|------|-------|
| Disha Chaturvedi | dchatur6@asu.edu |
| Zaina Mushtaq | zmushta1@asu.edu |
| Adarsh Mohan | amohan78@asu.edu |
| Priyanshu Patra | ppatra1@asu.edu |

---

## Description

MA-Rank is a multi-agent resume screening framework that converts resumes and job descriptions into structured Neo4j knowledge graph representations and generates transparent, explainable candidate rankings. It addresses the core limitations of keyword-based ATS systems — poor semantic matching, lack of transparency, and no uncertainty signaling — by using five specialized agents for extraction, normalization, graph writing, ranking, and consensus evaluation.

---

## Overview

Automated resume screening systems commonly fail due to rigid keyword matching, opaque scoring, and inconsistent document formatting. MA-Rank resolves these issues by structuring both resumes and job descriptions into a shared skill graph, enabling multi-dimensional candidate comparison with full explainability. The system outputs ranked candidate shortlists with matched skills, missing skills, experience and education fit, score breakdowns, uncertainty labels, and consensus reasoning.

---

## System Architecture

The pipeline consists of six phases, each handled by a dedicated agent:

| Phase | Agent | Responsibility |
|-------|-------|----------------|
| 1 | Parser Agent | Extracts name, email, experience, education, skills from resumes & JDs |
| 2 | Normalizer Agent | Deduplicates skills, maps aliases, standardizes education & experience |
| 3 | Knowledge Graph Writer | Writes nodes and relationships to Neo4j using MERGE queries |
| 4 | Matcher / Ranker Agent | Computes composite ranking score across 7 dimensions |
| 5 | Consensus Agent | Second-pass evaluation; flags uncertainty and recommends human review |
| 6 | Streamlit UI | Recruiter-facing interface for job upload, resume upload, and ranking |

### Neo4j Ontology

```
(:Category)-[:HAS_JOB]->(:Job)-[:REQUIRES]->(:Skill)<-[:HAS_SKILL]-(:Candidate)
```

---

## Ranking Score Formula

The composite candidate score is calculated as:

```
Score = 0.40 × skill_match_ratio
      + 0.15 × jaccard_score
      + 0.15 × weighted_skill_score
      + 0.10 × semantic_score        (TF-IDF Cosine Similarity)
      + 0.10 × experience_score
      + 0.05 × education_score
      + 0.05 × guttman_score         (domain-specific skill hierarchy)
```

Uncertainty is calculated separately based on skill coverage, missing skills, education evidence, and experience signals.

---

## Dataset

| Dataset | Source | Size |
|---------|--------|------|
| LinkedIn Job Postings | Kaggle | 922,000+ job postings across 11 CSV files |
| Resume Dataset | Kaggle | 2,484 resumes with extracted text |

After filtering to **sales** and **technology** domains:
- **1,534 jobs** (868 sales, 666 technology)
- **500 augmented resumes**

---

## Evaluation Results

### Baseline Ranking Comparison (AI/ML Developer @ Hykmann Technologies)

| Method | Precision@5 | Precision@10 | nDCG@10 | MRR |
|--------|-------------|--------------|---------|-----|
| Keyword Overlap | 1.00 | 0.70 | 1.0000 | 1.00 |
| BM25 | 0.40 | 0.50 | 0.9368 | 1.00 |
| **MA-Rank** | **1.00** | **0.80** | **0.9743** | **1.00** |

MA-Rank produced the strongest top-10 shortlist with 8 out of 10 candidates judged relevant.

### Skill Deterioration Test

| Candidate | Rank | Score | Skill Match | Experience Fit | Uncertainty |
|-----------|------|-------|-------------|----------------|-------------|
| Original | 1 | 3.761 | 40% | 20% | Medium |
| Deteriorated | 7 | 2.237 | 20% | 0% | High |

MA-Rank correctly penalized the weaker profile, dropping it from rank 1 to rank 7 with an elevated uncertainty label.

---

## Repository Structure

```
├── data/
│   ├── job_postings/          # Cleaned LinkedIn job postings (sales & tech)
│   └── resumes/               # Augmented resume dataset
├── agents/
│   ├── parser_agent.py        # Resume and JD extraction
│   ├── normalizer_agent.py    # Skill normalization and alias mapping
│   ├── graph_writer_agent.py  # Neo4j MERGE-based graph writing
│   ├── ranker_agent.py        # Composite scoring and uncertainty
│   └── consensus_agent.py    # Second-pass evaluation and risk flagging
├── app/
│   └── streamlit_app.py       # Recruiter-facing Streamlit UI
├── notebooks/                 # Evaluation and analysis notebooks
└── README.md
```

---

## Requirements

```bash
pip install pandas numpy streamlit neo4j sentence-transformers scikit-learn scipy pyxlsb
```

A running **Neo4j** instance is required. Configure connection credentials in your environment or a `.env` file:

```
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password
```

---

## Key Features

- **Semantic skill normalization** with alias dictionaries (cosine similarity threshold: 0.85)
- **Incremental graph updates** — new jobs and resumes added without full rebuild
- **Must-have vs. nice-to-have skill** separation from job descriptions
- **Uncertainty scoring** to flag borderline candidates for human review
- **Dual-agent consensus** to guard against single-model bias
- **Streamlit UI** with job uploader, resume uploader, and ranked candidate view

---

## Limitations

- Evaluation currently covers a single job posting — broader benchmarking needed
- Manual relevance labeling introduces subjectivity; inter-annotator agreement not yet measured
- Scope limited to sales and technology domains

---

## Future Work

- Expand evaluation across more job categories and domains
- Increase relevance-labeled test cases with actual hiring outcome data
- Explore bias mitigation strategies in the extraction and ranking pipeline

---

## References

1. Lo et al. (2025). AI Hiring with LLMs: A Context-Aware and Explainable Multi-Agent Framework. *CVPRW*. doi:10.1109/CVPRW67362.2025.00402
2. Bhattacharya & Verbert (2025). Let's Get You Hired. *arXiv*. doi:10.48550/arXiv.2505.20312
3. Dharmendra et al. (2025). NLP-Powered Resume Screening and Ranking System. *ICDT*. doi:10.1109/ICDT63985.2025.10986338
4. Tanberk et al. (2023). Resume Matching Framework via Ranking and Sorting. *UBMK*. doi:10.1109/UBMK59864.2023.10286605
5. Cenikj et al. (2021). Skills NER for Creating a Skill Inventory. *IEEE Big Data*. doi:10.1109/BigData52589.2021.9671435
6. Upadhyay et al. (2021). Explainable Job-Posting Recommendations Using Knowledge Graphs. *IEEE SMC*. doi:10.1109/SMC52423.2021.9658757
7. Leela et al. (2025). Skill-Based Recommender System using Knowledge Graphs. *ICMNWC*. doi:10.1109/ICMNWC66779.2025.11354307
8. Li et al. (2021). Resume Information Extraction Using BERT-BiLSTM-CRF. *ICCT*. doi:10.1109/ICCT52962.2021.9657937
9. Zhang et al. (2022). Are Male Candidates Better than Females? Debiasing BERT Resume Retrieval. *IEEE SMC*. doi:10.1109/SMC53654.2022.9945184
10. Petrican et al. (2017). Ontology-based Skill Matching Algorithms. *ICCP*. doi:10.1109/ICCP.2017.8117005

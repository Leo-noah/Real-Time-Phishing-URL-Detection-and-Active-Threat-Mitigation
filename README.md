Real-Time Phishing URL Detection and Active Threat Mitigation

A Firefox extension that checks clicked links before normal navigation, combines XGBoost predictions with domain trust and security rules, and responds with **SAFE**, **WARNING**, or **DANGEROUS**.

Developed as an academic cybersecurity prototype.

| Project information | Details |
| --- | --- |
| Selected model | Optimized XGBoost-1 |
| Final decision thresholds | 20% / 80% |

## Overview

Phishing links can lead users to fake websites that attempt to steal credentials and personal information. Manual URL checkers require users to inspect a link before opening it, which adds an extra step to everyday browsing.

This project brings URL checking into the browser workflow. The Firefox extension intercepts supported link clicks, checks a local blacklist, and sends unknown URLs to a locally running Flask backend. The backend extracts 20 URL and domain features, runs the trained XGBoost model, and combines the prediction with domain trust and security rules to produce a final danger score.

The extension then allows navigation, displays a warning, or blocks the destination. Warning and blocking pages also explain the risk to support cybersecurity awareness.

## Key Features

- **Browser-level interception:** Checks supported clicked URLs before the destination opens.
- **Local blacklist:** Recognizes previously blocked domains without repeating model analysis.
- **Machine learning analysis:** Uses XGBoost with 20 URL and domain features.
- **Hybrid scoring:** Combines model predictions with domain trust and security rules.
- **Three response levels:** Allows low-risk links, warns about uncertain links, and blocks high-risk links.
- **Security awareness pages:** Explains suspicious or dangerous results to users.
- **Local analysis:** Runs the prediction backend locally without a cloud-hosted URL classification service.

## How the System Works

```text
User Clicks a URL
        ↓
Firefox Extension Intercepts the Click
        ↓
Local Blacklist Check
        ├── Known Blocked Domain → Block Immediately
        │
        └── Unknown Domain
                ↓
          Local Flask Backend
                ↓
          Extract 20 URL Features
                ↓
          XGBoost Prediction
                +
          Domain Trust / Security Rules
                ↓
          Final Danger Score
                ↓
          ≤ 20%           SAFE
          > 20% and < 80% WARNING
          ≥ 80%           DANGEROUS
                ↓
          Allow / Warn / Block
```

Newly detected dangerous domains can be stored in the local blacklist for subsequent checks.

## Security Decisions

| Result | Final danger score | Browser response |
| --- | --- | --- |
| SAFE | Less than or equal to 20% | Continues normal navigation. |
| WARNING | Greater than 20% and less than 80% | Displays a warning with **Go Back Safely** and **Proceed Anyway** options. |
| DANGEROUS | Greater than or equal to 80% | Displays a blocking page with a safe-return option and no proceed option. |

A blacklist match takes the immediate blocking path. The score thresholds apply to URLs sent for backend analysis. A SAFE result is a system classification, not a guarantee that a website is harmless.

## System Components

| Component | Responsibility |
| --- | --- |
| Firefox extension | Intercepts clicks, checks the blacklist, requests analysis, and applies the result. |
| Local storage | Keeps blocked domains using `browser.storage.local`. |
| Flask backend | Receives URLs and coordinates feature extraction, prediction, and scoring. |
| Feature extractor | Converts each URL into the 20 inputs expected by the trained model. |
| XGBoost model | Estimates malicious-URL probability from the extracted features. |
| Domain trust and security rules | Adjust the decision using domain reputation and suspicious URL indicators. |
| Warning and blocking interfaces | Show the destination, risk information, and available user actions. |

The prototype does not require user accounts or a traditional database server.

## URL Analysis

The report describes features and scoring indicators covering:

- URL length, domain length, and URL/domain entropy.
- Subdomains, digits, hyphens, and special characters.
- HTTPS usage and IP-based hostnames.
- Suspicious keywords, paths, and file extensions.
- Query parameters and redirect indicators.
- Brand-spoofing indicators and trusted-domain status.
- User-generated-content hosting domains.

These are categories of the reported analysis, rather than an exact ordered list of the 20 model inputs. Popular hosting platforms are treated separately because a trusted platform can still host malicious user-created content.

## Technology Stack

| Area | Technologies |
| --- | --- |
| Browser extension | Firefox WebExtension, JavaScript, HTML, CSS |
| Backend | Python, Flask |
| Machine learning | XGBoost, Scikit-learn |
| Data processing | Pandas, NumPy |
| URL processing | TLDExtract, `urllib.parse`, regular expressions |
| Model persistence | Joblib |
| Domain reputation data | Top-1M domain list |
| Development tools | JupyterLab, Visual Studio Code |
| Testing environment | Ubuntu 22.04 LTS in VirtualBox on Windows 11 |

## Dataset and Model Development

Three source datasets contained **1,473,168 URL records before preprocessing**. After cleaning conflicting and duplicate records, the report gives **1,449,532 URLs** for the baseline model, divided into:

- **1,159,625** training URLs.
- **289,907** testing URLs.

The training workflow used an 80/20 stratified split. Later experiments retrained the model with difficult false-positive and false-negative examples and higher sample weights.

The selected final model is **Optimized XGBoost-1**, saved as:

```text
FINAL_HARD_RETRAINED_PHISHING_MODEL.pkl
```

Optimized XGBoost-1 was chosen for its balance between detection and interruptions to legitimate browsing. The final hybrid configuration uses the **20% / 80% thresholds**.

## Reported Evaluation Results

The following figures come from the project report. They have not been independently reproduced for this README.

### Binary Model Comparison

The optimized models were compared on **858,347 unique URLs**: 325,129 safe URLs and 533,218 dangerous URLs.

| Metric | Optimized XGBoost-1 | Optimized XGBoost-2 |
| --- | --- | --- |
| Accuracy | 97.48% | 97.58% |
| Precision | 99.18% | 98.02% |
| Recall | 96.75% | 98.09% |
| F1-score | 97.95% | 98.05% |
| False-positive rate | 1.32% | 3.25% |
| False-negative rate | 3.25% | 1.91% |

The baseline model achieved **93.61% accuracy** on its separate baseline test split. Its score should not be treated as a direct same-test-set comparison with the optimized models.

### Final Hybrid Configuration: Optimized XGBoost-1 at 20% / 80%

| Metric | Reported result |
| --- | --- |
| Safe URLs incorrectly blocked | 0.15% |
| Dangerous URLs incorrectly allowed as SAFE | 0.63% |
| Dangerous URLs receiving WARNING or DANGEROUS | 99.37% |
| Overall warning rate | 28.14% |

**Security coverage includes both warnings and blocks.** It is not the percentage of dangerous URLs automatically blocked. The wider warning band reduces critical allow/block errors while asking users to review more uncertain links.

These are experimental results for the reported datasets and configuration. They do not establish performance on every unseen URL or browsing environment. Because hard-example retraining used earlier misclassifications, a fresh independent dataset remains important for assessing generalization.

## Local Setup Overview

The report documents the architecture and configuration process, but does not provide a complete reproducible repository installation guide. The following describes the required components; exact commands, backend filenames, ports, and paths should be confirmed against the source code.

1. **Prepare the environment.** Use Firefox and a compatible Python environment. The reported development and testing environment was Ubuntu 22.04 LTS inside VirtualBox.
2. **Install project dependencies.** Use the repository's dependency specification, if provided. The report lists Flask, XGBoost, Scikit-learn, Pandas, NumPy, TLDExtract, and Joblib.
3. **Prepare the model and domain data.** Ensure the backend can load `FINAL_HARD_RETRAINED_PHISHING_MODEL.pkl` and its trusted-domain data. Feature order and preprocessing must match model training.
4. **Start the local backend.** Use the project's actual backend entry point and confirm that the extension's configured API address matches it.
5. **Load the Firefox extension.** Use Firefox's temporary extension-loading workflow with the project's `manifest.json` during development.
6. **Verify the integration.** Confirm that links reach the backend, results trigger the intended interfaces, and stored blacklist entries bypass repeat model analysis. Use controlled test inputs in an isolated environment.

## Scope and Limitations

- This is an academic prototype focused on phishing indicators in URL structure and domain information.
- Predictions can produce false positives and false negatives.
- The system does not inspect webpage content, downloaded file contents, or background network behavior.
- The implementation was developed and tested mainly for Firefox; Chromium-based browser support is future work.
- Unknown URLs depend on a running local backend and the required model/data files.
- Detection quality depends on dataset coverage, feature extraction, model freshness, and scoring rules.
- Warning pages allow users to continue, so a warning does not guarantee prevention.
- Broader testing on unseen datasets, browsers, operating systems, and navigation patterns is still needed.

## Future Improvements

- Add support for Chrome, Edge, and Brave.
- Evaluate on fresh phishing datasets and improve domain reputation signals.
- Reduce unnecessary warnings while maintaining detection coverage.
- Explore browser-side inference to reduce backend setup requirements.
- Improve blacklist management, logging, and threat-history visibility.
- Add model updates and investigate operating-system security integration.

## What I Learned

This project gave me practical experience in browser extension development, Flask integration, URL feature engineering, XGBoost training, hard-example retraining, and security testing. It also showed me why a security tool needs to balance missed threats, false blocks, and warning fatigue rather than relying on accuracy alone.

## Project Report

**An Intelligent Real-Time URL Interception and Active Threat Mitigation System for Cybersecurity Awareness**  

This README summarizes the supplied project report, particularly its architecture, implementation, evaluation, final model selection, and limitations. Source code and runtime behavior were not inspected when preparing this version.

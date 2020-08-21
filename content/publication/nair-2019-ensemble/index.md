---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: An ensemble prediction model for train delays
subtitle: ''
summary: ''
authors:
- Rahul Nair
- Thanh Lam Hoang
- Marco Laumanns
- Bei Chen
- Randall Cogill
- Jácint Szabó
- Thomas Walter
tags: []
categories: []
date: '2019-01-01'
lastmod: 2020-08-21T10:43:12+01:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ''
  focal_point: ''
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: ["Deutsche Bahn"]
url_pdf: https://www.sciencedirect.com/science/article/pii/S0968090X18317984

publishDate: '2020-08-21T09:43:12.711624Z'
publication_types:
- 2
abstract: 'A large-scale ensemble prediction model to predict train delays is presented. The ensemble model uses a disparate set of models, two statistical and one simulation-based to generate forecasts of train delays. The first statistical model is a context-aware random forest that accounts for network traffic states, such as likely stretch conflicts and current headway’s, exogenous weather, event, and work zone information. The second model is a kernel regression that captures train-specific dynamics. A mesoscopic simulation model that accounts for travel and dwell time variations as well as inferred track occupation conflicts, train connections and rolling stock rotations, is additionally considered. The models have been used in a proof of concept to forecast delays for nationwide passenger services network of Deutsche Bahn, which operates roughly 25,000 trains daily in Germany. Results demonstrate a 25% improvement potential in forecast correctness (fraction of predictions within one minute) and 50% reduction in root mean squared errors compared to the published schedule. The paper describes the models along with the big data challenges that were addressed in data storage, feature and model building, and computation.'
publication: '*Transportation Research Part C: Emerging Technologies*'
---

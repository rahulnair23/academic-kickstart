---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: What Changed? Interpretable Model Comparison
subtitle: ''
summary: 'Addresses the problem of distinguishing two machine learning (ML) models built for the same task in a human-interpretable way'
authors:
- Rahul Nair
- Massimiliano Mattetti
- Elizabeth Daly
- Dennis Wei
- Öznur Alkan
- Yunfeng Zhang
tags: []
categories: []
date: '2021-01-01'
lastmod: 2021-10-26T15:44:13+01:00
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
projects: []
publishDate: '2021-10-26T14:44:12.476701Z'
publication_types:
- 1
abstract: 'We consider the problem of distinguishing two machine learning (ML) models built for the same task in a human-interpretable way. As models can fail or succeed in different ways, classical accuracy metrics may mask crucial qualitative differences. This problem arises in a few contexts. In business applications with periodically retrained models, an updated model may deviate from its predecessor for some segments without a change in overall accuracy. In automated ML systems, where several ML pipelines are generated, the top pipelines have comparable accuracy but may have more subtle differences. We present a method for interpretable comparison of binary classification models by approximating them with Boolean decision rules. We introduce stabilization conditions that allow for the two rule sets to be more directly comparable. A method is proposed to compare two rule sets based on their statistical and semantic similarity by solving assignment problems and highlighting changes. An empirical evaluation on several benchmark datasets illustrates the insights that may be obtained and shows that artificially induced changes can be reliably recovered by our method. '
url_pdf: https://www.ijcai.org/proceedings/2021/0393.pdf
publication: '*Proceedings of the Thirtieth International Joint Conference on Artificial
  Intelligence, $$IJCAI-21$$*'
---

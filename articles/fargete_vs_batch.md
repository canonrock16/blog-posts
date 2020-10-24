---
title: "AWS fargateとAWS Batchの違い"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

fargateはGPUインスタンスが使えないがセキュリティ設定が楽らしい。
BatchはGPUインスタンスが使えたり、選択できるコンピューティング環境の最大スペックがfargateよりも大きいため、深層学習を使用するような機械学習プロジェクトや、分散処理をガッツリやりたいようなプロジェクトで使うとよい。

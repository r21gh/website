---
title: Podセキュリティ標準の強制
weight: 40
---

<!-- overview -->

このページでは、[Podセキュリティの標準](/ja/docs/concepts/security/pod-security-standards)を強制する際のベストプラクティスの概要を説明します。

<!-- body -->

## ビルトインPodセキュリティアドミッションコントローラーの使用

{{< feature-state for_k8s_version="v1.25" state="stable" >}}

[Podセキュリティアドミッションコントローラー](/docs/reference/access-authn-authz/admission-controllers/#podsecurity)は、非推奨のPodSecurityPolicyを置き換えます。

### すべてのNamespaceに設定する

設定が全く無いNamespaceは、クラスターのセキュリティモデルにおいて重大な弱点とみなすべきです。
各Namespaceで発生するワークロードのタイプを時間をかけて分析し、Podセキュリティ標準を参照しながら、それぞれに適切なレベルを決定することを推奨します。
また、ラベルのないNamespaceは、まだ評価されていないことだけを示すべきです。

すべてのNamespaceのワークロードが同じセキュリティ要件を持つというシナリオでは、PodSecurityラベルを一括適用する方法を[例](/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/#applying-to-all-namespaces)で説明しています。

### 最小特権の原則を採用する

理想的な世界では、すべてのNamespaceのPodが`restricted`ポリシーの要件を満たすでしょう。
しかし、ワークロードの中には正当な理由で昇格した特権を必要とするものもあるため、それは不可能であり、現実的でもありません。

- `privileged`ワークロードを許可するNamespaceは、適切なアクセス制御を確立し、実施すべきである。
- 最小権限のNamespaceで実行されるワークロードについては、そのワークロードのセキュリティ要件に関するドキュメントを整備する。可能であれば、それらの要件がどのように制約される可能性があるのかを考慮する。

### マルチモード戦略の採用

Podセキュリティアドミッションコントローラーの`audit`と`warn`モードを使用すると、既存のワークロードを破壊することなく、Podに関する重要なセキュリティインサイトを簡単に収集できます。

すべてのNamespaceでこれらのモードを有効にし、最終的に`enforce`したいレベルやバージョンに設定するのがよい方法です。
このフェーズで生成される警告と監査注釈は、その状態への指針となります。
ワークロード作成者が希望のレベルに収まるように変更することを期待している場合は、`warn`モードを有効にしてください。
監査ログを使用して、希望のレベルに収まるように変更を監視/推進することを期待している場合は、`audit`モードを有効にしてください。

`enforce`モードが希望通りの値に設定されている場合でも、これらのモードはいくつかの異なる方法で役立ちます。

- `warn`を`enforce`と同じレベルに設定すると、バリデーションを通過しないPod(またはPodテンプレートを持つリソース)を作成しようとしたときに、クライアントが警告を受け取るようになります。これにより、対象のリソースを更新して準拠させることができます。
- `enforce`を特定の最新バージョンではないに固定するNamespaceでは、`audit`と`warn`モードを`enforce`と同じレベルに設定するが、最新バージョンに対して設定することで、以前のバージョンでは許可されていたが、現在のベストプラクティスでは許可されていない設定を可視化することができます。

## サードパーティによる代替案

{{% thirdparty-content %}}

Kubernetesエコシステムでは、セキュリティプロファイルを強制するための他の選択肢も開発されています。

- [Kubewarden](https://github.com/kubewarden)
- [Kyverno](https://kyverno.io/policies/)
- [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper)

ビルトインソリューション(PodSecurityアドミッションコントローラーなど)とサードパーティツールのどちらを選ぶかは、あなたの状況次第です。
どのようなソリューションを評価する場合でも、サプライチェーンの信頼が非常に重要です。最終的には、前述のアプローチのどれを使っても、何もしないよりはましでしょう。
本ドキュメントは意訳になります。
また、動作の詳細な確認、関連機能の調査などをしていないため、解釈ミス・翻訳ミスが含まれる可能性があることをご了承ください。

---

---
layout: blog
title: 'Kubernetes 1.23: The Next Frontier'
date: 2021-12-07
slug: kubernetes-1-23-release-announcement
---

# Kubernetes 1.23: The Next Frontier

Original Post: https://kubernetes.io/blog/2021/12/07/kubernetes-1-23-release-announcement/

**Authors:** [Kubernetes 1.23 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.23/release-team.md)

We’re pleased to announce the release of Kubernetes 1.23, the last release of 2021!

2021年の最後のリリースであるKubernetes 1.23のリリースをアナウンスできることを嬉しく思います。

This release consists of 47 enhancements: 11 enhancements have graduated to stable, 17 enhancements are moving to beta, and 19 enhancements are entering alpha. Also, 1 feature has been deprecated.     

このリリースには47個のEnhancementがありました。11個がStableとなり、17個がBetaとなり、19個がAlphaとなりました。また、1個がDepracatedとなりました。

## Major Themes

### Deprecation of FlexVolume

FlexVolumeはDeprecatedとなりました。out-of-treeのCSIドライバーがKubernetesにおいてボリュームに書き込む推奨方法となります。詳細は[このドキュメント](https://github.com/kubernetes/community/blob/master/sig-storage/volume-plugin-faq.md#kubernetes-volume-plugin-faq-for-storage-vendors)を参照してください。FlexVolumeのメンテなーはCSIドライバーを実装し、FlexVolumeのユーザーをCSIへ移行させるようにしてください。FlexVolumeのユーザーは、自信のワークロードをCSIドライバーへ移行してください。

FlexVolume is deprecated. The out-of-tree CSI driver is the recommended way to write volume drivers in Kubernetes. See [this doc](https://github.com/kubernetes/community/blob/master/sig-storage/volume-plugin-faq.md#kubernetes-volume-plugin-faq-for-storage-vendors) for more information. Maintainers of FlexVolume drivers should implement a CSI driver and move users of FlexVolume to CSI. Users of FlexVolume should move their workloads to the CSI driver.

### Deprecation of klog specific flags

コードをシンプルにするため、Kubernetes 1.23 ではいくつかの[ログのためのフラグがDeprecatedとなりました。](https://kubernetes.io/docs/concepts/cluster-administration/system-logs/#klog) 将来的にそのためのコードは削除されるため、ユーザーはDeprecatedのフラグを代替手法に置き換え始める必要があります。

To simplify the code base, several [logging flags were marked as deprecated](https://kubernetes.io/docs/concepts/cluster-administration/system-logs/#klog) in Kubernetes 1.23. The code which implements them will be removed in a future release, so users of those need to start replacing the deprecated flags with some alternative solutions.

### Software Supply Chain SLSA Level 1 Compliance in the Kubernetes Release Process

Kubernetes リリースは、リリースプロセスに置いてステージングとリリースのフェーズを説明した来歴の証明ファイル(provenance attestation files)を生成するようになりました。1つのフェーズから次のフェーズに移行する時に作成物が検証されています。この最後のピースによって、[SLSA(サルサ) security framework](https://slsa.dev/) (Supply-chain Levels for Software Artifacts)のLevel 1に準拠するのに必要な作業を完了します。

Kubernetes releases now generate provenance attestation files describing the staging and release phases of the release process. Artifacts are now verified as they are handed over from one phase to the next. This final piece completes the work needed to comply with Level 1 of the [SLSA security framework](https://slsa.dev/) (Supply-chain Levels for Software Artifacts).

### IPv4/IPv6 Dual-stack Networking graduates to GA

[IPv4/IPv6 dual-stack networking](https://github.com/kubernetes/enhancements/tree/master/keps/sig-network/563-dual-stack) がGAとなります。1.21
からKubernetesクラスターはデュアルスタックをデフォルトでサポートしています。1.23では `IPv6DualStack` フィーチャーゲートは削除されました。デュアルスタックネットワーキングの使用は必須ではありません。クラスターはデュアルスタックをサポート可能ですが、PodsとServicesはデフォルトでシングルスタックのままです。デュアルスタックネットワーキングを使用するためには、KubernetesノードはIPv4とIPv6でルーティング可能なネットワークインターフェースが必要で、デュアルスタックで動作するCNIネットワークプラグインを使わなければなりません。さらに、Podsはデュアルスタックで設定されている必要があり、Servicesは`.spec.ipFamilyPolicy` フィールドに `PreferDualStack` か `RequireDualStack` を設定しなければなりません。

[IPv4/IPv6 dual-stack networking](https://github.com/kubernetes/enhancements/tree/master/keps/sig-network/563-dual-stack) graduates to GA. Since 1.21, Kubernetes clusters have been enabled to support dual-stack networking by default. In 1.23, the `IPv6DualStack` feature gate is removed. The use of dual-stack networking is not mandatory. Although clusters are enabled to support dual-stack networking, Pods and Services continue to default to single-stack. To use dual-stack networking Kubernetes nodes must have routable IPv4/IPv6 network interfaces, a dual-stack capable CNI network plugin must be used, Pods must be configured to be dual-stack and Services must have their `.spec.ipFamilyPolicy` field set to either `PreferDualStack` or `RequireDualStack`.

### HorizontalPodAutoscaler v2 graduates to GA

HorizontalPodAutoscaler の安定版API `autoscaling/v2` が1.23でGAとなりました。HorizontalPodAutoscaler の `autoscaling/v2beta2` APIはDeprecatedとなりました。

The HorizontalPodAutscaler `autoscaling/v2` stable API moved to GA in 1.23. The HorizontalPodAutoscaler `autoscaling/v2beta2` API has been deprecated.

### Generic Ephemeral Volume feature graduates to GA

Generic Ephemeral Volume 機能が1.23ではGAとなりました。この機能はダイナミックプロビジョニングをサポートする既存ストレージドライバーに、ボリュームのライフサイクルがPodに紐付いている一時ボリューム(Ephemeral Volume)として使用されます。ボリュームプロビジョニングのためのすべてのStorageClassのパラメータと、PersistentVolumeClaimでサポートされたすべて機能がサポートされます。

The generic ephemeral volume feature moved to GA in 1.23. This feature allows any existing storage driver that supports dynamic provisioning to be used as an ephemeral volume with the volume’s lifecycle bound to the Pod. All StorageClass parameters for volume provisioning and all features supported with PersistentVolumeClaims are supported.

### Skip Volume Ownership change graduates to GA

Podsのボリュームのパーミッションとオーナーシップの変更ポリシーを設定する機能が1.23ではGAとなりました。これによりユーザーはマウント時の再帰的なパーミッション変更をスキップし、Podの開始時間を高速化することを可能とします。

The feature to configure volume permission and ownership change policy for Pods moved to GA in 1.23. This allows users to skip recursive permission changes on mount and speeds up the pod start up time.

### Allow CSI drivers to opt-in to volume ownership and permission change graduates to GA

CSIドライバーが fsGroupベースのパーミッションのサポートを宣言する機能が1.23ではGAとなりました。

The feature to allow CSI Drivers to declare support for fsGroup based permissions graduates to GA in 1.23.

### PodSecurity graduates to Beta

[PodSecurity](https://kubernetes.io/docs/concepts/security/pod-security-admission/) がBetaとなりました。`PodSecurity` はDeprecatedとなっている `PodSecurityPolicy` のAdmission Controllerを置き換えます。 `PodSecurity` はNamespaceの中のPodsに対して、強制レベルをセットする特定のNamespaceのラベルに基づき Podのセキュリティ標準(Pod Security Standards)を強制するAdmission Controllerです。1.23では `PodSecurity` フィーチャーゲートはデフォルトで有効になっています。

[PodSecurity](https://kubernetes.io/docs/concepts/security/pod-security-admission/) moves to Beta. `PodSecurity` replaces the deprecated `PodSecurityPolicy` admission controller. `PodSecurity` is an admission controller that enforces Pod Security Standards on Pods in a Namespace based on specific namespace labels that set the enforcement level. In 1.23, the `PodSecurity` feature gate is enabled by default.

### Container Runtime Interface (CRI) v1 is default

Kubelet はCRI `v1` をサポートし、プロジェクト横断でデフォルトとなりました。
もし、コンテナーランタイムが `v1` APIをサポートしていない場合、Kuberntesは `v1alpha2` の実装へフォールバックします。エンドユーザーに対しては必要な中間のアクションはありません。`v1` と `v1alpha2` には実装上の相違点がないからです。`v1alpha2` は将来のKubernetesリリースの1つで `v1` を開発可能とするために削除されるでしょう。

The Kubelet now supports the CRI `v1` API, which is now the project-wide default. 
If a container runtime does not support the `v1` API, Kubernetes will fall back to the `v1alpha2` implementation. There is no intermediate action required by end-users, because `v1` and `v1alpha2` do not differ in their implementation. It is likely that `v1alpha2` will be removed in one of the future Kubernetes releases to be able to develop `v1`.

### Structured logging graduate to Beta

構造化されたログがBetaのマイルストーンに到達しました。kubeletとkube-sheculerのログメッセージの大部分はすでに変換されています。ユーザーは、JSON出力または構造化テキスト形式の解析を試して、ログ値の複数行の文字列の処理など、未解決の問題に対する可能な解決策についてフィードバックを提供することをお勧めします。

Structured logging reached its Beta milestone. Most log messages from kubelet and kube-scheduler have been converted. Users are encouraged to try out JSON output or parsing of the structured text format and provide feedback on possible solutions for the open issues, such as handling of multi-line strings in log values.

### Simplified Multi-point plugin configuration for scheduler

kube-scheduler は新しいシンプルな設定フィールドを追加しています。これによりプラグインは複数の拡張ポイントにおける有効化を1箇所で実施できます。新しい `multiPoint` プラグインフィールドは管理者のためにほとんどのスケジューラーの初期化を単純にすることを目的としています。`multiPoint`によって有効化されたプラグインは自動的にそれらが実装する個別の拡張ポイントに登録されます。例えば、スコアとフィルターの拡張が実装されているプラグインは、同時に2つが有効化できます。これは、プラグイン全体を個別の拡張ポイント設定を手動で編集すること無しに有効あるいは無効にできることを意味します。これらの拡張ポイントは、ほとんどのユーザーにとって無関係であるため、抽象化できるようになりました。

The kube-scheduler is adding a new, simplified config field for Plugins to allow multiple extension points to be enabled in one spot. The new `multiPoint` plugin field is intended to simplify most scheduler setups for administrators. Plugins that are enabled via `multiPoint` will automatically be registered for each individual extension point that they implement. For example, a plugin that implements Score and Filter extensions can be simultaneously enabled for both. This means entire plugins can be enabled and disabled without having to manually edit individual extension point settings. These extension points can now be abstracted away due to their irrelevance for most users.

### CSI Migration updates

CSIマイグレーションは、`kubernetes.io/gce-pd` や`kubernetes.io/aws-ebs`のような既存のin-treeのストレージプラグインと、関連するCSIドライバーの置き換えを可能にします。CSIマイグレーションが適切に動作する場合、Kubernetesのエンドユーザーは違いを感じないでしょう。マイグレーション後も、Kubernetesのユーザーは既存のインターフェースを使用してin-treeのストレージプラグインのすべての機能に依存し続ける可能性があります。
- CSIマイグレーション機能はデフォルトで有効化されていますが、1.23ではGCE PD、AWS EBS、AzureDiskではBetaのままです。
- CSIマイグレーションは、1.23ではCeph RBDとPortworxではAlphaとして導入されています。

CSI Migration enables the replacement of existing in-tree storage plugins such as `kubernetes.io/gce-pd` or `kubernetes.io/aws-ebs` with a corresponding CSI driver. 
If CSI Migration is working properly, Kubernetes end users shouldn’t notice a difference. 
After migration, Kubernetes users may continue to rely on all the functionality of in-tree storage plugins using the existing interface.
- CSI Migration feature is turned on by default but stays in Beta for GCE PD, AWS EBS, and Azure Disk in 1.23.
- CSI Migration is introduced as an Alpha feature for Ceph RBD and Portworx in 1.23.

### Expression language validation for CRD is alpha

CRDにおける表現言語(Expression Laungage)のバリデーションが1.23ではAlphaとなります。`CustomResourceValidationExpressions` フィーチャーゲートが有効の場合、カスタムリソースは[Common Expression Language (CEL)](https://github.com/google/cel-spec) によるバリデーションルールによって検証されます。

Expression language validation for CRD is in alpha starting in 1.23. If the `CustomResourceValidationExpressions` feature gate is enabled, custom resources will be validated by validation rules using the [Common Expression Language (CEL)](https://github.com/google/cel-spec).

### Server Side Field Validation is Alpha

1.23から `ServerSideFieldValidation` フィーチャーゲートが有効になっている場合、リクエスト時にユーザーが不明なフィールドや重複したフィールドを含むKubernetesオブジェクトを送信した時、サーバから警告を受け取ることになります。これまでは、不明なフィールドと最後の重複フィールドを除くすべてがサーバーによって削除されます。

このフィーチャーゲートの有効化によって、`fieldValidation` クエリーパラメータも導入されます。これにより、ユーザーは望むサーバーのふるまいをリクエストごとに指定することができます。`fieldValidation` クエリーパラメータに有効な値は次のとおりです。
- Ignore (フィーチャーゲートが無効のときのデフォルト値。1.23以前の不明フィールドのドロップ・無視の動作と同じ。)
- Warn (フィーチャーゲートが有効のときのデフォルト値)
- Strict (Invalid Requestエラー時にリクエストを失敗させる)

If the `ServerSideFieldValidation` feature gate is enabled starting 1.23, users will receive warnings from the server when they send Kubernetes objects in the request that contain unknown or duplicate fields. Previously unknown fields and all but the last duplicate fields would be dropped by the server.

With the feature gate enabled, we also introduce the `fieldValidation` query parameter so that users can specify the desired behavior of the server on a per request basis. Valid values for the `fieldValidation` query parameter are:

- Ignore (default when feature gate is disabled, same as pre-1.23 behavior of dropping/ignoring unkonwn fields)
- Warn (default when feature gate is enabled).
- Strict (this will fail the request with an Invalid Request error)

### OpenAPI v3 is Alpha

1.23では、`OpenAPIV3` フィーチャーゲートが有効の場合、ユーザーはすべてのKubernetesタイプに対して OepenAPI v3.0 仕様をリクエスト可能です。OpenAPI v3 は完全の透過性を目指しており、OpenAPI v2の公開時に削除される一連のフィールド(`default`, `nullable`, `oneOf`, `anyOf`)のサポートを含みます。分割された仕様はパフォーマンスと発見性の向上のためにKubernetesのグループバージョンごと(`$cluster/openapi/v3/apis/<group>/<version>`のエンドポイント)に公開されており、すべてのグループバージョンは `$cluster/openapi/v3` のパスで見られます。

If the `OpenAPIV3` feature gate is enabled starting 1.23, users will be able to request the OpenAPI v3.0 spec for all Kubernetes types. OpenAPI v3 aims to be fully transparent and includes support for a set of fields that are dropped when publishing OpenAPI v2: `default`, `nullable`, `oneOf`, `anyOf`. A separate spec is published per Kubernetes group version (at the `$cluster/openapi/v3/apis/<group>/<version>` endpoint) for improved performance and discovery, for all group versions can be found at the `$cluster/openapi/v3` path.

## Other Updates

### Graduated to Stable

- [IPv4/IPv6 Dual-Stack Support](https://github.com/kubernetes/enhancements/issues/563)
- [Skip Volume Ownership Change](https://github.com/kubernetes/enhancements/issues/695)
- [TTL After Finished Controller](https://github.com/kubernetes/enhancements/issues/592)
- [Config FSGroup Policy in CSI Driver object](https://github.com/kubernetes/enhancements/issues/1682)
- [Generic Ephemeral Inline Volumes](https://github.com/kubernetes/enhancements/issues/1698)
- [Defend Against Logging Secrets via Static Analysis](https://github.com/kubernetes/enhancements/issues/1933)
- [Namespace Scoped Ingress Class Parameters](https://github.com/kubernetes/enhancements/issues/2365)
- [Reducing Kubernetes Build Maintenance](https://github.com/kubernetes/enhancements/issues/2420)
- [Graduate HPA API to GA](https://github.com/kubernetes/enhancements/issues/2702)


### Major Changes

- [Priority and Fairness for API Server Requests](https://github.com/kubernetes/enhancements/issues/1040)

### Release Notes

Kubernetes 1.23 リリースの完全な詳細はこちらの [リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.23.md) をチェックしてください。

Check out the full details of the Kubernetes 1.23 release in our [release notes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.23.md).

### Availability

Kubernetes 1.23 は[GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.23.0)からダウンロード可能です。Kubernetesを始めるには、こちらの[対話型チュートリアル](https://kubernetes.io/docs/tutorials/) でチェックしたり、[kind](https://kind.sigs.k8s.io/)によるDockerコンテナーの "ノード" でKubernetesクラスターをローカルで実行してください。[kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)を使って1.23を簡単にインストールすることも可能です。

Kubernetes 1.23 is available for download on [GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.23.0). To get started with Kubernetes, check out these [interactive tutorials](https://kubernetes.io/docs/tutorials/) or run local Kubernetes clusters using Docker container “nodes” with [kind](https://kind.sigs.k8s.io/). You can also easily install 1.23 using [kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/). 

### Release Team

このリリースは、すべてのKubernetesリリースに技術的なコンテンツ、ドキュメンテーション、コードおよび他のコンポーネントのホストを提供するためのチームとして集まった非常に熱心な個人の集まりによって可能になりました。

リリースサイクルを成功に導いてくれたリリースリードであるRey Lejanoと、リリースチームの全員がお互いをサポートし、1.23リリースを提供するための一生懸命な働きに心から感謝します。

This release was made possible by a very dedicated group of individuals, who came together as a team to deliver technical content, documentation, code, and a host of other components that go into every Kubernetes release.

A huge thank you to the release lead Rey Lejano for leading us through a successful release cycle, and to everyone else on the release team for supporting each other, and working so hard to deliver the 1.23 release for the community.

### Release Theme and Logo 

**Kubernetes 1.23: The Next Frontier**

{{< figure src="/images/blog/2021-12-07-kubernetes-release-1.23/kubernetes-1.23.png" alt="" class="release-logo" >}}

"The Next Frontier" というテーマは1.23における新しい段階的な機能強化と、Kubernetesのスタートレックを参照した歴史、そしてリリースチームのコミュニティメンバーの成長を表しています。

"The Next Frontier" theme represents the new and graduated enhancements in 1.23, Kubernetes' history of Star Trek references, and the growth of community members in the release team.

Kubernetes はスタートレックを参照している歴史があります。GoogleにおけるKubernetesのオリジナルのコード名はProject 7で、これはスタートレック ヴォイジャーのSeven of Nineを参照しています。そしてもちろんBorgはKubernetesの前身の名前でした。"The Next Frontier"というテーマも続けてスタートレックを参照しています。"The Next Frontier" は2つのスタートレックのタイトルを合わせたもので、それは、スタートレックV: The Final Frontier と スタートレック the Next Generation です。

Kubernetes has a history of Star Trek references. The original codename for Kubernetes within Google is Project 7, a reference to Seven of Nine from Star Trek Voyager. And of course Borg was the name for the predecessor to Kubernetes. "The Next Frontier" theme continues the Star Trek references. "The Next Frontier" is a fusion of two Star Trek titles, Star Trek V: The Final Frontier and Star Trek the Next Generation.

"The Next Frontier" はSIG Releaseの憲章にも一行含まれており、"長期にわたるリリースプロセスをサポートするために、コミュニティメンバーの一貫したグループであること" と書かれています。リリースチームで、新しいリリースチームメンバーとともにオープンソースのフロンティアにはじめて貢献する多くの人のためにコミュニティを成長させます。

"The Next Frontier" represents a line in the SIG Release charter, "Ensure there is a consistent group of community members in place to support the release process across time." With each release team, we grow the community with new release team members and for many it's their first contribution in their open source frontier.



Reference: https://kubernetes.io/blog/2015/04/borg-predecessor-to-kubernetes/
Reference: https://github.com/kubernetes/community/blob/master/sig-release/charter.md

Kubernetes 1.23 のロゴももたスタートレックを参照しています。それぞれの星はKubernetesのロゴの舵(Helm)を表しています。船はリリースチームの集合的なチームワークを新たしています。

The Kubernetes 1.23 release logo continues with the theme's Star Trek reference. Every star is a helm from the Kubernetes logo. The ship represents the collective teamwork of the release team.

Rey Lejanoがロゴをデザインしました。

Rey Lejano designed the logo. 

### User Highlights

- [Findings of the latest CNCF End User Technology Radar](https://www.cncf.io/announcements/2021/09/22/cncf-end-user-technology-radar-provides-insights-into-devsecops/) はDevSecOps関連のテーマを扱っています。詳細と調査結果は[Radar Page](https://radar.cncf.io/) を参照してください。
- エンドユーザーのAegon Life Indiaが主要なデジタルサービス企業への変革を目指し、[コアプロセスを従来のモノリスからマイクロサービスベースアーキテクチャに移行した方法](https://www.cncf.io/case-studies/aegon-life-india/)について説明しています。
- 複数のクラウドネイティブプロジェクトを活用した[Seagateによるエッヂにおけるリアルタイム分析を実行するためのedgerX](https://www.cncf.io/case-studies/seagate/). 
- [ZambonがSparkFabrikと協力してクラウドネイティブ技術を使用して16のウェブサイトを開発し、一貫したブランドアイデンティティを維持しながらステークホルダーがコンテンツを更新できるようになった方法](https://www.cncf.io/case-studies/zambon/)を説明しています。
- Kubernetesを使い、[InfluxData はマルチクラウド、マルチリージョンのサービス可用性を約束できるようになりました。](https://www.cncf.io/case-studies/influxdata/) これは、3つの主要なクラウドプロバイダーにまたがる複数のグローバルクラスターへの単一アプリケーションとしてのInfluxDBのシームレスな配信を可能にする真のクラウド抽象化レイヤーの作成により実現しています。

- [Findings of the latest CNCF End User Technology Radar](https://www.cncf.io/announcements/2021/09/22/cncf-end-user-technology-radar-provides-insights-into-devsecops/) were themed around DevSecOps. Check out the [Radar Page](https://radar.cncf.io/) for the full details and findings.
- Learn about how [end user Aegon Life India migrated core processes from its traditional monolith to a microservice-based architecture](https://www.cncf.io/case-studies/aegon-life-india/) in its effort to transform into a leading digital service company.
- Utilizing multiple cloud native projects, [Seagate engineered edgerX to run Real-time Analytics at the Edge](https://www.cncf.io/case-studies/seagate/). 
- Check out how [Zambon worked with SparkFabrik to develop 16 websites, with cloud native technologies, to enable stakeholders to easily update content while maintaining a consistent brand identity](https://www.cncf.io/case-studies/zambon/).
- Using Kubernetes, [InfluxData was able to deliver on the promise of multi-cloud, multi-region service availability](https://www.cncf.io/case-studies/influxdata/) by creating a true cloud abstraction layer that allows for the seamless delivery of InfluxDB as a single application to multiple global clusters across three major cloud providers.

    
### Ecosystem Updates

- [KubeCon + CloudNativeCon NA 2021](https://www.cncf.io/events/kubecon-cloudnativecon-north-america-2021/) が2021年の10月にオンラインと現地の両方で開催されました。すべてのトークは確認したい人なら誰でも[現在オンデマンドで確認可能です!](https://www.youtube.com/playlist?list=PLj6h78yzYM2Nd1U4RMhv7v88fdiFqeYAP) 
- [Kubernetes and Cloud Native Essentials Training と KCNA 認定は現在一般に登録とスケジュールができるようになりました。](https://www.cncf.io/announcements/2021/11/18/kubernetes-and-cloud-native-essentials-training-and-kcna-certification-now-available/). 加えて、新しいオンライントレーニングコースの [Kubernetes and Cloud Native Essentials (LFS250)](https://www.cncf.io/announcements/2021/10/13/entry-level-kubernetes-certification-to-help-advance-cloud-careers/)がリリースされました。これによって、個人が入門レベルのクラウドの役割のための準備と、KCNA試験の準備を行うことができます。
- [the Inclusive Naming Initiativeより新しいリソースが使用可能になりました。](https://www.cncf.io/announcements/2021/10/13/inclusive-naming-initiative-announces-new-community-resources-for-a-more-inclusive-future/)それは、Inclusive Strategies for Open Source (LFC103) のコース、 言語評価フレームワーク(Language Evaluation Framework)、およびその実装パスを含みます。

    
### Project Velocity

[CNCF K8s DevStats project](https://k8s.devstats.cncf.io/d/12/dashboards?orgId=1&refresh=15m)では、Kubernetesやそのサブプロジェクトのベロシティに関する興味深いデータの数字を集約します。ここには個人のコントリビュータからコントリビューションしている企業の数まで含まれており、このエコシステムを進化させる努力の深さと広さを示しています。

The [CNCF K8s DevStats](https://k8s.devstats.cncf.io/d/12/dashboards?orgId=1&refresh=15m) project aggregates a number of interesting data points related to the velocity of Kubernetes and various sub-projects. This includes everything from individual contributions to the number of companies that are contributing, and is an illustration of the depth and breadth of effort that goes into evolving this ecosystem.

v1.23 リリースサイクルは16週間(08/23 - 12/07)でした。[1032の企業](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.22.0%20-%20now&var-metric=contributions) と [1084の個人](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.22.0%20-%20now&var-metric=contributions&var-repogroup_name=Kubernetes&var-country_name=All&var-companies=All&var-repo_name=kubernetes%2Fkubernetes)によるコントリビューションがありました。

In the v1.23 release cycle, which ran for 16 weeks (August 23 to December 7), we saw contributions from [1032 companies](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.22.0%20-%20now&var-metric=contributions) and [1084 individuals](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.22.0%20-%20now&var-metric=contributions&var-repogroup_name=Kubernetes&var-country_name=All&var-companies=All&var-repo_name=kubernetes%2Fkubernetes).

### Event Update

- [KubeCon + CloudNativeCon China 2021](https://www.lfasiallc.com/kubecon-cloudnativecon-open-source-summit-china/) が12/09 - 12/11に開催されます。 去年は開催されませんでしたが、今年はバーチャル開催で105のセッションが含まれます。イベントスケジュールを[ここからチェックしてください](https://www.lfasiallc.com/kubecon-cloudnativecon-open-source-summit-china/program/schedule/)。
- KubeCon + CloudNativeCon Europe 2022 が2022年05/04 - 05/07でスペインのバレンシアで開催されます!カンファレンスの情報や登録については[イベントのサイトから確認してください。](https://events.linuxfoundation.org/archive/2021/kubecon-cloudnativecon-europe/).
- Kubernetes Community Days がパキスタン、ブラジル、成都、オーストラリアで開催予定です。

### Upcoming Release Webinar

2022/01/04にKubernetes 1.23のリリースチームメンバーがこのリリースの主要な機能やアップグレード計画を助けるための非推奨、削除機能について学ぶために(リリースウェビナーに)参加します。詳細情報と登録のためには、CNCF Online Programsサイトの[イベントページ](https://community.cncf.io/e/mrey9h/) を参照してください。

Join members of the Kubernetes 1.23 release team on January 4, 2022 to learn about the major features of this release, as well as deprecations and removals to help plan for upgrades. For more information and registration, visit the [event page](https://community.cncf.io/e/mrey9h/) on the CNCF Online Programs site.

### Get Involved

Kubernetesに参加するための最も簡単な方法は、多くの[Special Interest Groups](https://github.com/kubernetes/community/blob/master/sig-list.md)(SIGs)の中で自分の興味のあるものに参加することです。
もしコミュニティと共有したいことがある場合は、週次の[コミュニティミーティング](https://github.com/kubernetes/community/tree/master/communication)に参加することも可能ですし、以下のチャンネルも活用してください。

The simplest way to get involved with Kubernetes is by joining one of the many [Special Interest Groups](https://github.com/kubernetes/community/blob/master/sig-list.md) (SIGs) that align with your interests. Have something you’d like to broadcast to the Kubernetes community? Share your voice at our weekly [community meeting](https://github.com/kubernetes/community/tree/master/communication), and through the channels below:

* [Kubernetes Contributors](https://www.kubernetes.dev/)でコントリビューションについての詳細を見つけられます。
* 最新の更新についてはTwitterの[@Kubernetesio](https://twitter.com/kubernetesio)をフォローしてください。
* コミュニティでの相談は[Discuss](https://discuss.kubernetes.io/)に参加してください。
* [Slack](http://slack.k8s.io/)のコミュニティに参加してください。
* あなたのKubernetesの[ストーリー](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)を共有してください。
* [blog](https://kubernetes.io/blog/)でKubernetesで起きてる出来事を読むことが出来ます。
* [Kubernetes Release Team](https://github.com/kubernetes/sig-release/tree/master/release-team)についての詳細を知ることが出来ます。

- Find out more about contributing to Kubernetes at the [Kubernetes Contributors](https://www.kubernetes.dev/) website
- Follow us on Twitter [@Kubernetesio](https://twitter.com/kubernetesio) for the latest updates
- Join the community discussion on [Discuss](https://discuss.kubernetes.io/)
- Join the community on [Slack](http://slack.k8s.io/)
- Post questions (or answer questions) on [Stack Overflow](http://stackoverflow.com/questions/tagged/kubernetes)
- Share your Kubernetes [story](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)
- Read more about what’s happening with Kubernetes on the [blog](https://kubernetes.io/blog/)
- Learn more about the [Kubernetes Release Team](https://github.com/kubernetes/sig-release/tree/master/release-team)


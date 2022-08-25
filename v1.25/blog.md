---
layout: blog
title: "Kubernetes v1.25: Combiner"
date: 2022-08-23
slug: kubernetes-v1-25-release
---

**Authors**: [Kubernetes 1.25 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.25/release-team.md)

　Announcing the release of Kubernetes v1.25!
Kubernetes v1.25のリリースをアナウンスします!

　This release includes a total of 40 enhancements. Fifteen of those enhancements are entering Alpha, ten are graduating to Beta, and thirteen are graduating to Stable. We also have two features being deprecated or removed.
このリリースは合計40個のEnhancementがあります。15個がAlphaとなり、10個がベータとなり、13個がStableとなります。また、2つの機能がDeprecatedあるいは削除されました。

## Release theme and logo 

**Kubernetes 1.25: Combiner**

{{< figure src="/images/blog/2022-08-23-kubernetes-1.25-release/kubernetes-1.25.png" alt="Combiner logo" class="release-logo" >}}

　The theme for Kubernetes v1.25 is _Combiner_.
Kubernetes v1.25のテーマは _(結集する人々)Combiner_ です。

　The Kubernetes project itself is made up of many, many individual components that, when combined, take the form of the project you see today. It is also built and maintained by many individuals, all of them with different skills, experiences, histories, and interests, who join forces not just as the release team but as the many SIGs that support the project and the community year-round.

Kubernetesプロジェクトはそれ自体が多くの、大変多くの個人によって作られています。そしてそれが結集されたとき、現在のプロジェクトの形となります。異なるスキル、経験、歴史的背景、興味を持つ多くの個人により作られ、メンテナンスされています。この人々は、リリースチームとしてだけではなく、年間を通してプロジェクトとコミュニティをサポートする多くのSIGとして力を集結させています。

　With this release we wish to honor the collaborative, open spirit that takes us from isolated developers, writers, and users spread around the globe to a combined force capable of changing the world. Kubernetes v1.25 includes a staggering 40 enhancements, none of which would exist without the incredible power we have when we work together.

このリリースで、我々はコラボレーティブでオープンな精神を敬いたいと思います。それは世界に散らばっている孤立した開発者、執筆者、ユーザーである我々を、世界を変えうる力へと導きます。Kubernetes v1.25は驚異的な40のEnhancementを含んでおり、これは我々が強力した時の信じられないほどの力がなければどれも実現しません。

　Inspired by our release lead's son, Albert Song, Kubernetes v1.25 is named for each and every one of you, no matter how you choose to contribute your unique power to the combined force that becomes Kubernetes.

我々のリリースリードの息子であるAlbert Songからインスパイアされ、Kubernetes v1.25はKubernetesを形作る結集された力に、あなた独自の力をどのように貢献することを選んだかは関係なく、あなた達一人ひとりを表す形で名前されました。

:memo: つまりそれがCombinerであるということですね。

## What's New (Major Themes)

### PodSecurityPolicy is removed; Pod Security Admission graduates to Stable {#pod-security-changes}

　PodSecurityPolicy was initially [deprecated in v1.21](/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/), and with the release of v1.25, it has been removed. The updates required to improve its usability would have introduced breaking changes, so it became necessary to remove it in favor of a more friendly replacement. That replacement is [Pod Security Admission](/docs/concepts/security/pod-security-admission/), which graduates to Stable with this release. If you are currently relying on PodSecurityPolicy, please follow the instructions for [migration to Pod Security Admission](/docs/tasks/configure-pod-container/migrate-from-psp/).

PodSecurityPolicyは始め[v1.21でDeprecated](/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/)となり、v1.25リリースで削除されました。ユーザビリティの向上に必要であった更新は、破壊的な変更を引き起こすため、より使いやすい仕組みを優先する形で削除が必要でした。代替する[Pod Security Admission](/docs/concepts/security/pod-security-admission/)は、今回のリリースでStableとなります。もし現在PodSecurityPolicyを使っている場合は、[Pod Security Admissionへの移行](/docs/tasks/configure-pod-container/migrate-from-psp/)手順を参照してください。

### Ephemeral Containers Graduate to Stable

　[Ephemeral Containers](/docs/concepts/workloads/pods/ephemeral-containers/) are containers that exist for only a limited time within an existing pod. This is particularly useful for troubleshooting when you need to examine another container but cannot use `kubectl exec` because that container has crashed or its image lacks debugging utilities. Ephemeral containers graduated to Beta in Kubernetes v1.23, and with this release, the feature graduates to Stable.

[Ephemeral Containers](/docs/concepts/workloads/pods/ephemeral-containers/)は既に存在するPodの中で限定的な時間だけ存在するためのコンテナです。これは他のコンテナを検査したくても、コンテナがクラッシュしていたりデバッグ用のユーティリティがコンテナ内に無いために`kubectl exec`を使えない場合のトラブルシューティングで特に便利です。Ephemeral ContainersはKubernetes v1.23でベータとなっており、今回のリリースでStableとなります。

### Support for cgroups v2 Graduates to Stable

　It has been more than two years since the Linux kernel cgroups v2 API was declared stable. With some distributions now defaulting to this API, Kubernetes must support it to continue operating on those distributions. cgroups v2 offers several improvements over cgroups v1, for more information see the [cgroups v2](https://kubernetes.io/docs/concepts/architecture/cgroups/) documentation. While cgroups v1 will continue to be supported, this enhancement puts us in a position to be ready for its eventual deprecation and replacement.

Linux Kernel cgroup v2 APIがStableとなってから2年以上が経っています。いくつかのディストリビューションでは現在デフォルトでこのAPIが使われており、Kubernetesはそれらのディストリビューションで運用を継続するためにサポートしなければなりません。cgroup v2はcgroup v1に対していくつかの改良点を提供します。詳細は[cgroups v2](https://kubernetes.io/docs/concepts/architecture/cgroups/)ドキュメントを参照してください。cgroup v1は引き続きサポートされますが、今回の拡張はcgroup v1の将来的な非推奨化と置き換えへに備えることができます。

### Improved Windows support

　- [Performance dashboards](http://perf-dash.k8s.io/#/?jobname=soak-tests-capz-windows-2019) added support for Windows
　- [Unit tests](https://github.com/kubernetes/kubernetes/issues/51540) added support for Windows
　- [Conformance tests](https://github.com/kubernetes/kubernetes/pull/108592) added support for Windows
　- New GitHub repository created for [Windows Operational Readiness](https://github.com/kubernetes-sigs/windows-operational-readiness)

- [Performance dashboards](http://perf-dash.k8s.io/#/?jobname=soak-tests-capz-windows-2019)にWindowsのサポートが追加されました。
- [Unit tests](https://github.com/kubernetes/kubernetes/issues/51540)にWindowsのサポートが追加されました。
- [Conformance tests](https://github.com/kubernetes/kubernetes/pull/108592)にWindowsのサポートが追加されました。
- [Windows Operational Readiness](https://github.com/kubernetes-sigs/windows-operational-readiness)のために新規にGitHubレポジトリが作歳されました。

### Moved container registry service from k8s.gcr.io to registry.k8s.io

　[Moving container registry from k8s.gcr.io to registry.k8s.io](https://github.com/kubernetes/kubernetes/pull/109938) got merged. For more details, see the [wiki page](https://github.com/kubernetes/k8s.io/wiki/New-Registry-url-for-Kubernetes-\(registry.k8s.io\)), [annoucement](https://groups.google.com/a/kubernetes.io/g/dev/c/DYZYNQ_A6_c/m/oD9_Q8Q9AAAJ) was sent to the kubernetes development mailing list.

[コンテナレジストリのk8s.gcr.ioからregistry.k8s.ioへの移動](https://github.com/kubernetes/kubernetes/pull/109938) が完了しました。詳細は、[Wikiページ](https://github.com/kubernetes/k8s.io/wiki/New-Registry-url-for-Kubernetes-\(registry.k8s.io\))やKubernetesの開発者メーリングリストに送られた[アナウンス](https://groups.google.com/a/kubernetes.io/g/dev/c/DYZYNQ_A6_c/m/oD9_Q8Q9AAAJ)を参照してください。

### Promoted SeccompDefault to Beta

　SeccompDefault promoted to beta, see the tutorial [Restrict a Container's Syscalls with seccomp](https://kubernetes.io/docs/tutorials/security/seccomp/#enable-the-use-of-runtimedefault-as-the-default-seccomp-profile-for-all-workloads) for more details.

SeccompDefaultがベータとなりました。詳細は[Restrict a Container's Syscalls with seccomp](https://kubernetes.io/docs/tutorials/security/seccomp/#enable-the-use-of-runtimedefault-as-the-default-seccomp-profile-for-all-workloads)のチュートリアルを参照してください。

### Promoted endPort in Network Policy to Stable

　Promoted `endPort` in [Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/#targeting-a-range-of-ports) to GA. Network Policy providers that support `endPort` field now can use it to specify a range of ports to apply a Network Policy. Previously, each Network Policy could only target a single port.

[Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/#targeting-a-range-of-ports) の `endPort`がGAとなりました。`endPort`フィールドをサポートするNetwork Policyプロバイダでポートの範囲を指定してNetwork Policyを適用するために使うことができます。これまでは、1つのポートだけを対象としてNetwork Policyがそれぞれ必要でした。

　Please be aware that `endPort` field **must be supported** by the Network Policy provider. If your provider does not support `endPort`, and this field is specified in a Network Policy, the Network Policy will be created covering only the port field (single port).

`endPort`フィールドはNetwork Policyプロバイダによって**サポートされていなければならない**ことに注意してください。もしあなたのプロバイダが`endPort`をサポートしておらず、Network Policyにこのフィールドが指定されていた場合、Network Policyは(1つの)portフィールドだけをカバーする形で作られます。

:memo: `port` と `endPort` の2つを指定していた場合でもサポートされてない場合は `port` だけが解釈され、結果的に1つのポートだけでNetwork Policyが作成されるということのようです。

### Promoted Local Ephemeral Storage Capacity Isolation to Stable
　The [Local Ephemeral Storage Capacity Isolation](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/361-local-ephemeral-storage-isolation) feature moved to GA. This was introduced as alpha in 1.8, moved to beta in 1.10, and it is now a stable feature. It provides support for capacity isolation of local ephemeral storage between pods, such as `EmptyDir`, so that a pod can be hard limited in its consumption of shared resources by evicting Pods if its consumption of local ephemeral storage exceeds that limit.

[Local Ephemeral Storage Capacity Isolation](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/361-local-ephemeral-storage-isolation) がGAとなりました。これは1.8でAlphaとして導入され、1.10でBetaとなったもので、今回Stableとなりました。これは`EmpltyDir`のようなPod間のローカル一時ストレージ容量の分離機能を提供するものです。Podがローカル一時ストレージの制限を超えて消費している場合、そのPodはEvictされることによって共有リソースの消費を厳しく制限されます。

### Promoted core CSI Migration to Stable

　[CSI Migration](https://kubernetes.io/blog/2021/12/10/storage-in-tree-to-csi-migration-status-update/#quick-recap-what-is-csi-migration-and-why-migrate) is an ongoing effort that SIG Storage has been working on for a few releases. The goal is to move in-tree volume plugins to out-of-tree CSI drivers and eventually remove the in-tree volume plugins. The [core CSI Migration](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/625-csi-migration) feature moved to GA. CSI Migration for GCE PD and AWS EBS also moved to GA. CSI Migration for vSphere remains in beta (but is on by default). CSI Migration for Portworx moved to Beta (but is off-by-default).

[CSI Migration](https://kubernetes.io/blog/2021/12/10/storage-in-tree-to-csi-migration-status-update/#quick-recap-what-is-csi-migration-and-why-migrate)はSIG Storageはこれまで数リリースにわたり作業しており、現在も実施中です。ゴールはin-treeのボリュームプラグインをout-of-treeのCSIドライバへ移行することで、最終的にはin-treeのボリュームプラグインを削除することです。[コアのCSI Migration](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/625-csi-migration)機能はGAとなりました。GCE PDとAWS EBSのCSI MigrationもまたGAとなりました。vSphereのCSI Migrationはベータのままです(ただしデフォルトでオン)。PortworxのCSI Migrationはベータとなりました(ただしデフォルトでオフ)。

### Promoted CSI Ephemeral Volume to Stable

　The [CSI Ephemeral Volume](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/596-csi-inline-volumes) feature allows CSI volumes to be specified directly in the pod specification for ephemeral use cases. They can be used to inject arbitrary states, such as configuration, secrets, identity, variables or similar information, directly inside pods using a mounted volume. This was initially introduced in 1.15 as an alpha feature, and it moved to GA. This feature is used by some CSI drivers such as the [secret-store CSI driver](https://github.com/kubernetes-sigs/secrets-store-csi-driver).

[CSI Ephemeral Volume](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/596-csi-inline-volumes)は一時的に利用するユースケースでCSIボリュームをPod定義時に直接指定することができる機能です。構成、シークレット、ID、変数や他の同様の情報などのような任意の状態を、ボリュームをマウントすることでPod内に直接差し込むために使用できます。これは1.15でAlpha機能として初めに導入され、GAとなりました。この機能は[secret-store CSIドライバ](https://github.com/kubernetes-sigs/secrets-store-csi-driver)のようないくつかのCSIドライバで使用されています。

### Promoted CRD Validation Expression Language to Beta

　[CRD Validation Expression Language](https://github.com/kubernetes/enhancements/blob/master/keps/sig-api-machinery/2876-crd-validation-expression-language/README.md) is promoted to beta, which makes it possible to declare how custom resources are validated using the [Common Expression Language (CEL)](https://github.com/google/cel-spec). Please see the [validation rules](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#validation-rules) guide.

[CRD Validation Expression Language](https://github.com/kubernetes/enhancements/blob/master/keps/sig-api-machinery/2876-crd-validation-expression-language/README.md)がBetaとなりました。これは [Common Expression Language (CEL)](https://github.com/google/cel-spec)を使ってカスタムリソースがどのようにバリデートされるかを定義することを可能にします。[バリデーションルール](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#validation-rules)のガイドを参照してください。

### Promoted Server Side Unknown Field Validation to Beta

　Promoted the `ServerSideFieldValidation` feature gate to beta (on by default). This allows optionally triggering schema validation on the API server that errors when unknown fields are detected. This allows the removal of client-side validation from kubectl while maintaining the same core functionality of erroring out on requests that contain unknown or invalid fields.

`ServerSideFieldValidation`のフィーチャーゲートがBetaとなりました(デフォルトでオン)。これはAPIサーバで不明なフィールドを検出したときにエラーとなるようにスキーマのバリデーションをオプションで指定できます。これはkubectlによるクライアントサイドのバリデーションを削除することを可能とし、同時に不明あるいは無効なフィールドを含むリクエスト時にエラーとする同じコア機能を維持することができます。

###  Introduced KMS v2 API

　Introduce KMS v2alpha1 API to add performance, rotation, and observability improvements. Encrypt data at rest (ie Kubernetes `Secrets`) with DEK using AES-GCM instead of AES-CBC for kms data encryption. No user action is required. Reads with AES-GCM and AES-CBC will continue to be allowed. See the guide [Using a KMS provider for data encryption](https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/) for more information.

パフォーマンス、ローテーションそして可観測性の向上のため、KMS v2alpha1 APIが導入されました。保存データ(つまりKubernetesの`Secret`)を、KMSデータ暗号化のためにAES-CBCではなくAES-GCMを使ってDEKにより暗号化します。ユーザのアクションは必要ありません。AES-GCMとAES-CBCによる読み込みは引き続き可能です。詳細は[Using a KMS provider for data encryption](https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/)のガイドを参照してください。

## Other Updates

### Graduations to Stable

　This release includes a total of thirteen enhancements promoted to stable: 
このリリースでは合計13個のEnhancementがStableとなりました。

* [Ephemeral Containers](https://github.com/kubernetes/enhancements/issues/277)
* [Local Ephemeral Storage Resource Management](https://github.com/kubernetes/enhancements/issues/361)
* [CSI Ephemeral Volumes](https://github.com/kubernetes/enhancements/issues/596)
* [CSI Migration - Core](https://github.com/kubernetes/enhancements/issues/625) 
* [Graduate the kube-scheduler ComponentConfig to GA](https://github.com/kubernetes/enhancements/issues/785)
* [CSI Migration - AWS](https://github.com/kubernetes/enhancements/issues/1487)
* [CSI Migration - GCE](https://github.com/kubernetes/enhancements/issues/1488) 
* [DaemonSets Support MaxSurge](https://github.com/kubernetes/enhancements/issues/1591)
* [NetworkPolicy Port Range](https://github.com/kubernetes/enhancements/issues/2079)
* [cgroups v2](https://github.com/kubernetes/enhancements/issues/2254)
* [Pod Security Admission](https://github.com/kubernetes/enhancements/issues/2579)
* [Add `minReadySeconds` to Statefulsets](https://github.com/kubernetes/enhancements/issues/2599)
* [Identify Windows pods at API admission level authoritatively](https://github.com/kubernetes/enhancements/issues/2802)

### Deprecations and Removals

　Two features were [deprecated or removed](/blog/2022/08/04/upcoming-changes-in-kubernetes-1-25/) from Kubernetes with this release.

このリリースではKubernetesから2つの機能が[非推奨あるいは削除](/blog/2022/08/04/upcoming-changes-in-kubernetes-1-25/)されました。

* [PodSecurityPolicy is removed](https://github.com/kubernetes/enhancements/issues/5)
* [GlusterFS plugin deprecated from available in-tree drivers](https://github.com/kubernetes/enhancements/issues/3446)

### Release Notes

　The complete details of the Kubernetes v1.25 release are available in our [release notes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.25.md).
Kubernetes v1.25リリースの完全な内容は、我々の[リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.25.md)で参照できます。

### Availability

Kubernetes v1.25 is available for download on [GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.25.0).
To get started with Kubernetes, check out these [interactive tutorials](/docs/tutorials/) or run local 
Kubernetes clusters using containers as “nodes”, with [kind](https://kind.sigs.k8s.io/).
You can also easily install 1.25 using [kubeadm](/docs/setup/independent/create-cluster-kubeadm/).


Kubernetes v1.25 は[GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.25.0)からダウンロード可能です。
Kubernetesを始めるには、こちらの[対話型チュートリアル](/docs/tutorials/)でチェックしたり、[kind](https://kind.sigs.k8s.io/)によりコンテナを使ってローカルのKubernetesクラスターを "ノード" として実行してください。
[kubeadm](/docs/setup/independent/create-cluster-kubeadm/)を使って1.25を簡単にインストールすることも可能です。


### Release Team

　Kubernetes is only possible with the support, commitment, and hard work of its community. Each release team is made up of dedicated community volunteers who work together to build the many pieces that, when combined, make up the Kubernetes releases you rely on. This requires the specialized skills of people from all corners of our community, from the code itself to its documentation and project management.

Kubernetesはコミュニティのサポート、コミットメントそしてハードな仕事によってのみ実現可能です。それぞれのリリースチームは献身的なコミュニティのボランティアメンバーによって構成されており、これらのメンバーが協力して多くの部分を構築し、結集したときに信頼できるKubernetesリリースを構成します。これにはコードそのものからドキュメントやプロジェクト管理に至るまで、私達のコミュニティの隅々にわたる人々の専門的なスキルが必要です。

　We would like to thank the entire release team for the hours spent hard at work to ensure we deliver a solid Kubernetes v1.25 release for our community. Every one of you had a part to play in building this, and you all executed beautifully. We would like to extend special thanks to our fearless release lead, Cici Huang, for all she did to guarantee we had what we needed to succeed.
コミュニティにKubernetes v1.25リリースを確実に提供するために多大な時間を費やしてくれたリリースチーム全体に感謝します。皆さん一人ひとりがこれを構築する役割を担っており、見事に実行しました。恐れを知らないリリースリードであるCici Huangには、私たちが成功するために必要なものが揃っていることを彼女が確実にしてくれたことに特に感謝しています。

### User Highlights

* Finleap Connect operates in a highly regulated environment. [In 2019, they had five months to implement mutual TLS (mTLS) across all services in their clusters for their business code to comply with the new European PSD2 payment directive](https://www.cncf.io/case-studies/finleap-connect/).
* PNC sought to develop a way to ensure new code would meet security standards and audit compliance requirements automatically—replacing the cumbersome 30-day manual process they had in place. Using Knative, [PNC developed internal tools to automatically check new code and changes to existing code](https://www.cncf.io/case-studies/pnc-bank/).
* Nexxiot needed highly-reliable, secure, performant, and cost efficient Kubernetes clusters. [They turned to Cilium as the CNI to lock down their clusters and enable resilient networking with reliable day two operations](https://www.cncf.io/case-studies/nexxiot/).
* Because the process of creating cyber insurance policies is a complicated multi-step process, At-Bay sought to improve operations by using asynchronous message-based communication patterns/facilities. [They determined that Dapr fulfilled its desired list of requirements and much more](https://www.cncf.io/case-studies/at-bay/). 

### Ecosystem Updates

* KubeCon + CloudNativeCon North America 2022 will take place in Detroit, Michigan from 24 – 28 October 2022! You can find more information about the conference and registration on the [event site](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/).
* KubeDay event series kicks off with KubeDay Japan December 7! Register or submit a proposal on the [event site](https://events.linuxfoundation.org/kubeday-japan/)
* In the [2021 Cloud Native Survey](https://www.cncf.io/announcements/2022/02/10/cncf-sees-record-kubernetes-and-container-adoption-in-2021-cloud-native-survey/), the CNCF saw record Kubernetes and container adoption. Take a look at the [results of the survey](https://www.cncf.io/reports/cncf-annual-survey-2021/). 

### Project Velocity

The [CNCF K8s DevStats](https://k8s.devstats.cncf.io/d/12/dashboards?orgId=1&refresh=15m) project 
aggregates a number of interesting data points related to the velocity of Kubernetes and various 
sub-projects. This includes everything from individual contributions to the number of companies that 
are contributing, and is an illustration of the depth and breadth of effort that goes into evolving this ecosystem.

In the v1.24 release cycle, which [ran for 15 weeks](https://github.com/kubernetes/sig-release/tree/master/releases/release-1.25) (May 23 to August 23), we saw contributions from [1065 companies](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.24.0%20-%20v1.25.0&var-metric=contributions) and [1620 individuals](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.24.0%20-%20v1.25.0&var-metric=contributions&var-repogroup_name=Kubernetes&var-country_name=All&var-companies=All&var-repo_name=kubernetes%2Fkubernetes).

## Upcoming Release Webinar

Join members of the Kubernetes v1.25 release team on Thursday September 22, 2022 10am – 11am PT to learn about 
the major features of this release, as well as deprecations and removals to help plan for upgrades. 
For more information and registration, visit the [event page](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-v125-release/).

## Get Involved

The simplest way to get involved with Kubernetes is by joining one of the many [Special Interest Groups](https://github.com/kubernetes/community/blob/master/sig-list.md) (SIGs) that align with your interests. 
Have something you’d like to broadcast to the Kubernetes community? Share your voice at our weekly [community meeting](https://github.com/kubernetes/community/tree/master/communication), and through the channels below:

* Find out more about contributing to Kubernetes at the [Kubernetes Contributors](https://www.kubernetes.dev/) website
* Follow us on Twitter [@Kubernetesio](https://twitter.com/kubernetesio) for the latest updates
* Join the community discussion on [Discuss](https://discuss.kubernetes.io/)
* Join the community on [Slack](http://slack.k8s.io/)
* Post questions (or answer questions) on [Server Fault](https://serverfault.com/questions/tagged/kubernetes).
* Share your Kubernetes [story](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)
* Read more about what’s happening with Kubernetes on the [blog](https://kubernetes.io/blog/)
* Learn more about the [Kubernetes Release Team](https://github.com/kubernetes/sig-release/tree/master/release-team)
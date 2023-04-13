本ドキュメントは意訳になります。
また、動作の詳細な確認、関連機能の調査などをしていないため、解釈ミス・翻訳ミスが含まれる可能性があることをご了承ください。

原文: https://kubernetes.io/blog/2023/04/11/kubernetes-v1-27-release/

---

**Authors**: [Kubernetes v1.27 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.27/release-team.md)

2023年最初のリリースとなるKubernetes v1.27のリリースを発表します！

このリリースは、60の機能拡張で構成されています。そのうち18の機能拡張がAlphaに、29の機能拡張がBetaに、そして13の機能拡張がStableに移行します。


## Release theme and logo

**Kubernetes v1.27: Chill Vibes**

Kubernetes v1.27のテーマは*穏やかな雰囲気*です。

{{< figure src="/images/blog/2023-04-11-kubernetes-1.27-blog/kubernetes-1.27.png" alt="Kubernetes 1.27 Chill Vibes logo" class="release-logo" >}}


少しばかばかしくはありますが、今回のリリースではこのテーマをひらめくのに役立つ重要な動きがいくつかありました。一般的なKubernetesのリリースサイクルでは、機能を含め続けるために必要な期限がいくつかあります。もし、ある機能がこれらの期限に間に合わなかった場合、例外処理があります。これらの例外を処理することは、リリースではごく普通のことです。しかし、v1.27では、機能拡張の凍結後、一度も例外処理を要求されず、記憶に残る最初のリリースとなりました。リリースが進むにつれて、私たちの誰もが慣れ親しんでいるよりもはるかに穏やかな状況が続いています。

今回、私たちがより落ち着いたリリースを楽しむことができたのには特別な理由があります。それはリリースを管理する方法を改善するために、裏方で人々が行った作業です。コミュニティのためにより良いものを作ろうとする人々の努力を称えるのがこのテーマなのです。

このロゴを作成してくれた[Britnee Laverack](https://www.instagram.com/artsyfie/)に特別な感謝を捧げます。Britneeは、[Kubernetes 1.24: Stargazer](https://kubernetes.io/blog/2022/05/03/kubernetes-1-24-release-announcement/#release-theme-and-logo)のロゴもデザインしています。

# What's New (Major Themes)

## Freeze `k8s.gcr.io` image registry

古いイメージレジストリである [k8s.gcr.io](https://cloud.google.com/container-registry/) を、数ヶ月前から一般に利用できるようになった [registry.k8s.io](https://github.com/kubernetes/registry.k8s.io) に置き換えます。Kubernetesプロジェクトは、コミュニティによって完全に制御された `registry.k8s.io` イメージレジストリを作成・運営しています。
これにより、古いレジストリ `k8s.gcr.io` は凍結され、今後Kubernetesおよび関連するサブプロジェクト向けの画像が古いレジストリに公開されることはないでしょう。

この変更はコントリビュータにとってどのような意味を持ちますか？

* サブプロジェクトのメンテナであれば、新しいレジストリを使用するためにマニフェストと Helm チャートを更新する必要があります。詳細については、この [プロジェクト](https://github.com/kubernetes-sigs/community-images) をチェックしてください。

この変更はエンドユーザーにとってどのような意味を持ちますか？

* Kubernetes `v1.27` のリリースは `k8s.gcr.io` レジストリに公開されない予定です。

* 4月以降、`v1.24`, `v1.25`, `v1.26` のパッチリリースは旧レジストリに公開されなくなります。

* v1.25 から、デフォルトのイメージレジストリは `registry.k8s.io` に設定されました。この値は kubeadm と kubelet で上書き可能ですが、`k8s.gcr.io` に設定すると、4月以降の新しいリリースでは古いレジストリに存在しないため、失敗します。

* クラスタの信頼性を高め、コミュニティが所有するレジストリへの依存を取り除きたい場合、または外部トラフィックが制限されているネットワークでKubernetesを実行している場合は、ローカルイメージレジストリのミラーの運用を検討する必要があります。クラウドベンダーによっては、このためのホスティングソリューションを提供している場合があります。


## `SeccompDefault` graduates to stable

seccompプロファイルのデフォルト設定を使用するには、使用する各ノードで `--seccomp-default` [command line flag](/docs/reference/command-line-tools-reference/kubelet) を有効にして kubelet を実行する必要があります。
有効にすると、kubeletは`Unconfined`（seccomp無効）モードを使用せず、コンテナランタイムが定義する`RuntimeDefault` seccompプロファイルをデフォルトで使用します。デフォルトのプロファイルは、ワークロードの機能を維持しながら、強力なセキュリティデフォルトのセットを提供することを目的としています。デフォルトのプロファイルは、コンテナランタイムとそのリリースバージョンによって異なる可能性があります。

可能なアップグレードとダウングレードの戦略に関する詳細情報は、関連するKubernetes Enhancement Proposal (KEP)に記載されています： [Enable seccomp by default](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/2413-seccomp-by-default)。

## Mutable scheduling directives for Jobs graduates to GA

これはv1.22で導入され、ベータ版としてスタートしましたが、現在は安定版となっています。並列ジョブはほとんどの場合で、ポッドを同じゾーンで動作させるか、GPUモデルxかyのどちらかで動作させてその両方を混在させないといった制約が必要です。`suspend`フィールドは、そのようなセマンティクスを実現するための最初のステップです。`suspend`フィールドは、カスタムキューコントローラがジョブをいつ開始すべきかを決定することを可能にします。しかし、ジョブのサスペンドが解除されると、カスタムキューコントローラは、ジョブのポッドが実際にどこに配置するか影響を与えることができません。

この機能により、Jobのスケジューリングディレクティブを開始前に更新することができ、カスタムキューコントローラが
Podの配置に影響を与えることができると同時に、実際のPodからノードへの割り当てをkube-schedulerにオフロードすることができます。
これは、過去に一度もサスペンドされたことのないサスペンドジョブに対してのみ許可されます。
ジョブのPodテンプレートで更新可能なフィールドは、node affinity、node selector、tolerations、labels、
annotations、[scheduling gates](/docs/concepts/scheduling-eviction/pod-scheduling-readiness/) があります。
詳しくはKEPをご覧ください：
[Allow updating scheduling directives of jobs](https://github.com/kubernetes/enhancements/tree/master/keps/sig-scheduling/2926-job-mutable-scheduling-directives)。


## DownwardAPIHugePages graduates to stable 

Kubernetes v1.20で、`requests.hugepages-<pagesize>` と `limits.hugepages-<pagesize>` が[downward API](/docs/concepts/workloads/pods/downward-api/) に追加されていました。これは、cpu、メモリ、エフェメラルストレージなどの他のリソースと一貫性を持っています。
この機能は、このリリースでgraduatedとなりました。詳細はKEPに記載されています：
[Downward API HugePages](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/2053-downward-api-hugepages)をご覧ください。

## Pod Scheduling Readiness goes to beta 

Podを作成すると、スケジューリングの準備が整います。Kubernetesのスケジューラは、保留中のすべてのPodを配置するノードを見つけるために適切な評価を行います。しかし、実際のケースでは、一部のPodが長期間にわたって _必須リソースが不足している_ 状態に留まる場合があります。このようなPodは、スケジューラ（と、Cluster Autoscalerのようなダウンストリームの統合など）を不必要な方法で混乱させます。

Podの `.spec.schedulingGates` を指定/削除することで、Podがスケジューリングの対象となる準備が整うタイミングを制御することができます。

`schedulingGates`フィールドは文字列のリストを含み、それぞれの文字列リテラルは、Podがスケジューリング可能とみなされる前に満たされなければならない基準として認識されます。このフィールドは、Podが作成されたときにのみ初期化することができます（クライアント、あるいはAdmissionによる変更のタイミング）。作成後、各schedulingGateは任意の順序で削除することができますが、新しいschedulingGateを追加することはできません。

## Node log access via Kubernetes API

この機能は、クラスタ管理者がサービスログを要求できるようにすることで、ノード上で実行されているサービスの問題をデバッグするのに役立ちます。この機能を使用するには、そのノードで `NodeLogQuery` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/) が有効になっており、kubelet 設定オプション `enableSystemLogHandler` と `enableSystemLogQuery` が両方とも true に設定されていることを確認してください。
Linuxでは、サービスログがjournald経由で利用可能であることを想定しています。Windowsでは、サービスログがアプリケーションログプロバイダーで利用可能であると仮定しています。また、それぞれLinuxでは`/var/log/`、Windowsでは`C:\log`ディレクトリからログを取得することができます。

クラスタ管理者は、クラスタの全ノード、またはそのサブセットでこのアルファ機能を試すことができます。

## ReadWriteOncePod PersistentVolume access mode goes to beta 

Kuberentes `v1.22` では、[PersistentVolumes](/docs/concepts/storage/persistent-volumes/#persistent-volumes) (PV) と [PersistentVolumeClaims](/docs/concepts/storage/persistent-volumeclaims) (PVCs) に対して新しいアクセスモード `ReadWriteOncePod` を追加しました。このアクセスモードでは、クラスター内の単一のポッドにボリュームアクセスを制限し、一度に1つのポッドのみがボリュームに書き込めるようにします。これは、ストレージへの単一の書き込みアクセスを必要とするステートフルなワークロードに特に有用です。

ReadWriteOncePodのベータ版は、ReadWriteOncePod PVCを使用するの[scheduler preemption](/docs/concepts/scheduling-eviction/pod-priority-preemption/)をサポートします。
Scheduler preemptionにより、優先度の高いポッドが優先度の低いポッドを先取りすることができます。例えば、`ReadWriteOncePod`のPVCを持つポッド（A）がスケジュールされているとき、同じPVCを使用している別のPod（B）が見つかり、Pod（A）の方が優先度が高い場合、スケジューラは`Unschedulable`ステータスを返してPod（B）の先取りを試みます。
より詳しいコンテキストについては、KEP: [ReadWriteOncePod PersistentVolume AccessMode](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/2485-read-write-once-pod-pv-access-mode)を参照してください。


## Respect PodTopologySpread after rolling upgrades

`matchLabelKeys`はPodのラベルキーのリストであり、Podの拡散を計算するために使用されます。このキーは、Podのラベルから値を検索するために使用されます。これらのキーと値のラベルは `labelSelector` と AND して、入力されるポッドに対して拡散を計算する対象となる既存のPodグループを選択するために使用されます。Podラベルに存在しないキーは無視されます。null または空のリストは、`labelSelector` に対してのみマッチすることを意味します。

`matchLabelKeys`を使用すると、ユーザーは異なるリビジョン間で `pod.spec` を更新する必要がありません。コントローラ/オペレータは、同じ `label` キーにリビジョンごとに異なる値を設定するだけです。スケジューラは `matchLabelKeys` に基づいて自動的にその値を設定します。例えば、ユーザーがDeploymentを使用する場合、Deploymentコントローラによって自動的に追加される `pod-template-hash` をキーとするラベルを使用して、1つのDeploymentの異なるリビジョンを区別することができます。


## Faster SELinux volume relabeling using mounts

本リリースでは、Podが使用するボリュームにSELinuxラベルを適用する方法がベータ版を卒業しました。この機能は、ボリューム上の各ファイルを再帰的に変更する代わりに、正しいSELinuxラベルを持つボリュームをマウントすることで、コンテナの起動を高速化するものです。SELinuxをサポートするLinuxカーネルでは、ボリュームの最初のマウント時に、`-o context=`マウントオプションを使用して、ボリューム全体にSELinuxラベルを設定することができます。この方法では、ボリューム全体を再帰的に処理せずに、一定の時間ですべてのファイルに与えられたラベルが割り当てられることになります。

マウントオプションの `context` は、バインドマウントや既にマウントされているボリュームの再マウントには適用することができません。
CSIストレージでは、CSIドライバがボリュームの最初のマウントを行うので、実際にマウントするのはCSIドライバでなければなりません。
CSIDriver オブジェクトに新しいフィールド `SELinuxMount` を追加し、これによってドライバが `-o context` マウントオプションをサポートしているかどうかをアナウンスします。

KubernetesがPodのSELinuxラベルを把握しており、**かつ**Podのボリュームを担当するCSIドライバがPodのボリュームの`SELinuxMount: true` を把握しており、**かつ** ボリュームのアクセスモードが `ReadWriteOncePod` の場合は以下のようになります。
CSI ドライバにマウントオプション `context=` でボリュームをマウントするよう要求します。**そして** 実行時にボリュームのコンテンツを再ラベルしないようにします（すべてのファイルがすでに正しいラベルを持つため）。
これについては、KEP: [Speed up SELinux volume relabeling using mounts](https://github.com/kubernetes/enhancements/tree/master/keps/sig-storage/1710-selinux-relabeling) から詳しい情報を入手してください。

## Robust VolumeManager reconstruction goes to beta

これは、ボリューム・マネージャーのリファクタリングで、kubelet起動時に既存のボリュームをマウントする際の追加情報を入力する方法についてのものです。
一般的に、これによりボリュームのクリーンアップがより堅牢になります。
ノードで `NewVolumeManagerReconstruction` フィーチャーゲートを有効にすると、kubelet 起動時にマウントされたボリュームの検出が強化されます。

Kubernetes v1.25以前では、kubeletはkubelet起動中にマウントされたボリュームを発見するために異なるデフォルト動作を使用しました。このフィーチャーゲートを無効にすると（デフォルトで有効になっています）、レガシーな検出動作が選択されます。

Kubernetes v1.25とv1.26では、この動作の切り替えは`SELinuxMountReadWriteOncePod`フィーチャーゲートの一部となっていました。

## Mutable Pod Scheduling Directives goes to beta

これにより、スケジューリング準備ゲートでブロックされたPodを、より制約の多いノードアフィニティ/セレクタに変更させることができます。また、外部リソースコントローラにポッドの配置に影響を与える能力を与えると同時に、実際のポッドとノードの割り当てをkube-schedulerにオフロードします。

これは、Kubernetesにスケジューリング機能を追加する新しいパターンを提供するものです。 具体的には、kube-schedulerがサポートしていない機能を実装する軽量スケジューラを構築しつつ、既存のkube-schedulerが対応する全てのアップストリーム機能を使うことができ、Podとノードの紐付けることができます。カスタム機能がスケジュールプラグインを実装する必要がなく、カスタムkube-schedulerバイナリの再ビルドと保守が必要な場合は、このパターンが好ましいでしょう。

## Feature graduations and deprecations in Kubernetes v1.27
### Graduations to stable

このリリースには、Stableに昇格した合計9つの機能拡張が含まれています：

* [Default container annotation that to be used by kubectl](https://github.com/kubernetes/enhancements/issues/2227)
* [TimeZone support in CronJob](https://github.com/kubernetes/enhancements/issues/3140)
* [Expose metrics about resource requests and limits that represent the pod model](https://github.com/kubernetes/enhancements/issues/1748)
* [Server Side Unknown Field Validation](https://github.com/kubernetes/enhancements/issues/2885)
* [Node Topology Manager](https://github.com/kubernetes/enhancements/issues/693)
* [Add gRPC probe to Pod.Spec.Container.{Liveness,Readiness,Startup} Probe](https://github.com/kubernetes/enhancements/issues/2727)
* [Add configurable grace period to probes](https://github.com/kubernetes/enhancements/issues/2238)
* [OpenAPI v3](https://github.com/kubernetes/enhancements/issues/2896)
* [Stay on supported Go versions](https://github.com/kubernetes/enhancements/issues/3744)

### Deprecations and removals

このリリースでは、いくつかの削除が行われました：

* [Removal of `storage.k8s.io/v1beta1` from CSIStorageCapacity](https://github.com/kubernetes/kubernetes/pull/108445)
* [Removal of support for deprecated seccomp annotations](https://github.com/kubernetes/kubernetes/pull/114947)
* [Removal of `--master-service-namespace` command line argument](https://github.com/kubernetes/kubernetes/pull/112797)
* [Removal of the `ControllerManagerLeaderMigration` feature gate](https://github.com/kubernetes/kubernetes/pull/113534)
* [Removal of `--enable-taint-manager` command line argument](https://github.com/kubernetes/kubernetes/pull/111411)
* [Removal of `--pod-eviction-timeout` command line argument](https://github.com/kubernetes/kubernetes/pull/113710)
* [Removal of the `CSI Migration` feature gate](https://github.com/kubernetes/kubernetes/pull/110410)
* [Removal of `CSIInlineVolume` feature gate](https://github.com/kubernetes/kubernetes/pull/111258)
* [Removal of `EphemeralContainers` feature gate](https://github.com/kubernetes/kubernetes/pull/111402)
* [Removal of `LocalStorageCapacityIsolation` feature gate](https://github.com/kubernetes/kubernetes/pull/111513)
* [Removal of `NetworkPolicyEndPort` feature gate](https://github.com/kubernetes/kubernetes/pull/110868)
* [Removal of `StatefulSetMinReadySeconds` feature gate](https://github.com/kubernetes/kubernetes/pull/110896)
* [Removal of `IdentifyPodOS` feature gate](https://github.com/kubernetes/kubernetes/pull/111229)
* [Removal of `DaemonSetUpdateSurge` feature gate](https://github.com/kubernetes/kubernetes/pull/111194)

## Release notes

Kubernetes v1.27のリリースの詳細については、我々の[リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.27.md)でご確認いただけます。

## Availability

Kubernetes v1.27は[GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.27.0)でダウンロード可能です。Kubernetesを始めるには、[minikube](https://minikube.sigs.k8s.io/docs/)、[kind](https://kind.sigs.k8s.io/)などを使用して、ローカルのKubernetesクラスタを実行することができます。また、[kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)を使って、v1.27を簡単にインストールすることができます。

## Release team

Kubernetesは、そのコミュニティのサポート、コミットメント、そしてハードワークによってのみ開発されています。各リリースチームは、献身的なコミュニティのボランティアで構成され、皆さんが信頼するKubernetesのリリースを構成する多くの部分を一緒に構築しています。そのためには、コードそのものからドキュメンテーション、プロジェクト管理まで、コミュニティの隅々から専門的なスキルを持った人々が必要です。

スムーズで成功したリリースサイクルを導いてくれたリリースリーダーのXander Grzywinskiと、コミュニティのためにv1.27リリースを作成するためにお互いをサポートし、懸命に働いてくれたリリースチームの全メンバーに特に感謝します。

## Ecosystem updates

* KubeCon + CloudNativeCon Europe 2023は、2023年4月17日から21日までオランダのアムステルダムで開催されます！カンファレンスの詳細や参加登録は[イベントサイト](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/)でご確認ください。
* cdCon + GitOpsConは、2023年5月8日と9日にカナダのバンクーバーで開催される予定です！ カンファレンスの詳細と登録は、[イベントサイト](https://events.linuxfoundation.org/cdcon-gitopscon/)で確認できます。

## Project velocity

[CNCF K8s DevStats](https://k8s.devstats.cncf.io/d/12/dashboards?orgId=1&refresh=15m)プロジェクトは、Kubernetesとさまざまなサブプロジェクトの速度に関連する多くの興味深いデータを集約しています。これには、個人の貢献から貢献する企業の数までが含まれ、このエコシステムを進化させるための努力の深さと幅を示すものです。

[14週間](https://github.com/kubernetes/sig-release/tree/master/releases/release-1.27)(1月9日～4月11日)のv1.27リリースサイクルでは、[1020社](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.26.0%20-%20now&var-metric=contributions)と[1603人の個人](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.26.0%20-%20now&var-metric=contributions&var-repogroup_name=Kubernetes&var-repo_name=kubernetes%2Fkubernetes&var-country_name=All&var-companies=All)からの貢献がありました。

## Upcoming release webinar

2023年4月14日（金）午前10時（PDT）より、Kubernetes v1.27リリースチームのメンバーと一緒に、このリリースの主要機能、およびアップグレードの計画に役立つ非推奨事項や削除事項について学びましょう。詳細および登録は、CNCF Online Programsサイトの[イベントページ](https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-v127-release/)をご覧ください。

## Get Involved

Kubernetesに参加する最も簡単な方法は、あなたの興味に沿った多くの[Special Interest Groups](https://github.com/kubernetes/community/blob/master/sig-list.md) (SIGs)のうちの1つに参加することです。

Kubernetesコミュニティに発信したいことがある場合は、毎週開催される[コミュニティ・ミーティング](https://github.com/kubernetes/community/tree/master/communication)や、以下のチャンネルであなたの声をシェアしてください：

* [Kubernetes Contributors website](https://www.kubernetes.dev/)で、Kubernetesへの貢献についてもっと知ることができます。

* 最新のアップデートについては、Twitter [@Kubernetesio](https://twitter.com/kubernetesio) で私たちをフォローしてください。

* [Discuss](https://discuss.kubernetes.io/)でコミュニティの議論に参加してください。

* [Slack](https://communityinviter.com/apps/kubernetes/community)でコミュニティに参加してください。

* [Server Fault](https://serverfault.com/questions/tagged/kubernetes) に質問を投稿してください（または質問に答える）。

* あなたのKubernetesストーリーを[Share](https://docs.google.com/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)してください。

* [ブログ](https://kubernetes.io/blog/)でKubernetesで何が起こっているのかについてもっと読んでください。

* [Kubernetes Release Team](https://github.com/kubernetes/sig-release/tree/master/release-team)についてもっと知ることができます。
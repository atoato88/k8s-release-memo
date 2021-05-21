本ドキュメントは意訳になります。
また、動作の詳細な確認、関連機能の調査などをしていないため、解釈ミス・翻訳ミスが含まれる可能性があることをご了承ください。

---

# Kubernetes Blog より

Original Post: https://kubernetes.io/blog/2021/04/08/kubernetes-1-21-release-announcement/

# Kubernetes 1.21: Power to the Community

2021年最初のリリースであるKubernetes 1.21を発表できて大変嬉しいです。
今回のリリースでは51の機能拡張(enhancements)があり、13個がStable、16個がBeta、20個がAlphaとなりました。また、2個が廃止予定(Deprecated)となりました。

リリースチームのオーナーシップのプロセスが変更され、定期的にコミュニティに情報を問い合わせるような形の同期したコミュニケーションから変わり、今回からコミュニティが機能やブログなどを提供してリリースに加えられるようになりました。
これによりコラボレーションやコミュニティを横断するチームワークが増え、最近の中でもっとも機能が多いリリースとなりました。

:pen: これまで以上にコミュニティ側が主導する活動へ変わったことで今までよりもやりやすくなったというニュアンスだと思います。"Power to the Community"の意味合いにつながるのだと思います。

# Major Themes
## CronJobs Graduate to Stable!

[CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)は1.8の頃にBetaとなりましたが、広く使われた結果1.21ではついにStableとなりました。

CronJobsはバックアップやレポート生成などの一般的なスケジュールのアクションを行うための手段となります。それぞれのタスクは無期限に繰り返し(1日1回、1週間、1ヶ月など)実行されます。この期間の中で実行する時間を指定します。

## Immutable Secrets and ConfigMaps

[ImmutableSecrets](https://kubernetes.io/docs/concepts/configuration/secret/#secret-immutable)と[Immutable ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/#configmap-immutable)として新規のフィールドを追加しました。これがセットされた場合、変更されようとしたときにその変更を却下します。SecretsとConfigMapsはデフォルトでは変更可能な設定でPodが活用します。一方、変更可能なSecretesやConfigMapsは設定の変更の不備により問題を引き起こすこともあります。

変更を不可とすることにより、アプリケーションの設定が変更されないことを明確に出来ます。変更したい場合は新たな独立した名前のSecretやConfigMapを作り直し、そのリソースを使うPodを作る必要があります。また、これらの変更不可のリソースはスケール時のメリットもあります。なぜなら変更の確認をAPIサーバに対してポーリングする必要がないからです。

この機能は1.21でStableとなりました。

## IPv4/IPv6 dual-stack support

IPアドレスはクラスタ運用者や管理者が利用するリソースで、使い切らないようにする必要があります。特にパブリックなIPv4アドレスは現在希少なものです。デュアルスタックサポートはPodやServiceに対するネイティブIPv6ルーティングを提供します。一方で必要に応じてクラスターがIPv4による通信も可能です。デュアルスタックのクラスターネットワークはワークロードのスケール時の制限を改善する可能性があります。

デュアルスタックサポートはPod、ServiceそしてNodeがIPv4とIPv6のアドレスを使える手段となります。1.21では[デュアルスタックネットワーキング](https://kubernetes.io/docs/concepts/services-networking/dual-stack/)はAlphaからBetaとなり、デフォルトで有効となります。

## Graceful Node Shutdown

[Graceful Node Shutdown](https://kubernetes.io/docs/concepts/architecture/nodes/#graceful-node-shutdown) はBetaとなりより多くのユーザ／グループが利用可能となりました。この大変利便性の高い機能はKubeletがノードのシャットダウンを検知可能となり、Gracefulにノード上のPodを終了させることが出来ます。

現在はノードがシャットダウンする時、Podは通常の終了するためのライフサイクルをたどることが出来ないためGracefulなシャットダウンは出来ませんでした。このことは異なる処理における多くの問題を引き起こします。今後はKubeletは差し迫ったシステムシャットダウンをSystemdを通して検知でき、稼働中のPodに可能な限りGracefulに処理を終了することを伝えることが出来ます。

## PersistentVolume Health Monitor 

PVはアプリケーションが共通的なローカルのファイルベースストレージとして使われます。PVは様々な使い方がされており、ストレージバックエンドの再書き込みを必要としないアプリケーションマイグレーションも手助けします。

1.21ではボリュームの正常性についてPVをモニタリングし、異常な場合はそれに応じたマークをつける機能をAlpha機能として持ちます。ワークロードは正常性を確認し、異常なボリュームへの書き込みや読み出しからデータを保護することが可能となります。

## Reducing Kubernetes Build Maintenance

これまでKubernetesは複数のビルドシステムでメンテナンスされていました。新コントリビュータと現行のコントリビュータにとって、このことはしばしば摩擦と複雑性の原因となっていました。

前回のリリースサイクルを通して、ビルドプロセスをシンプルにするための多くの対処が実施され、ネイティブGo言語によるビルドツールが標準化されました。このことはより広範囲のコミュニティメンテナンスに力を与え、新コントリビュータが参加する際の障壁を低くします。

# Major Changes
## PodSecurityPolicy Deprecation 

1.21ではPodSecurityPolicyがDeprecatedとなります。Kubernetesの全てのFeature Deprecationと同様に、今後数リリースにおいては利用可能であり全てが機能します。BetaだったPodSecurityPolicyは1.25で削除予定です。

代わりとなるものとして、Podの特権を制限する新しいビルドイン機構を開発しています。"PSP Replacement Policy" というタイトルでこれらの作業を行っています。PodSecurityPolicyのユースケースをカバーし利用者の使い勝手とメンテナンス性を大幅に向上させる計画です。詳細は[PodSecurityPolicy Deprecation: Past, Present, and Future](https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future)を読んでください。

## TopologyKeys Deprecation

Serviceに存在する`topologyKeys`フィールドがDeprecatedとなりました。このフィールドをつかっていた全てのコンポーネント機能はAlphaでした。そしてDeprecatedとなりました。私達は`topologyKeys`をtopology-aware hintsと呼ばれるtopology-awareルーティングを実装するための方法に変更しました。Topology-aware hintsは1.21ではAlphaです。詳細は[Topology Aware Hints](https://kubernetes.io/docs/concepts/services-networking/service-topology/)で読めます。関連[KEP](https://github.com/kubernetes/enhancements/blob/master/keps/sig-network/2433-topology-aware-hints/README.md)は今回の変更のコンテキストを説明しています。

# Other Updates
## Graduated to Stable
- [EndpointSlice](https://github.com/kubernetes/enhancements/issues/752)
- [Add sysctl support](https://github.com/kubernetes/enhancements/issues/34)
- [PodDisruptionBudgets](https://github.com/kubernetes/enhancements/issues/85)

## Notable Feature Updates
- [External client-go credential providers](https://github.com/kubernetes/enhancements/issues/541) - beta in 1.21
- [Structured logging](https://github.com/kubernetes/enhancements/issues/1602) - graduating to beta in 1.22
- [TTL after finish cleanup for Jobs and Pods](https://github.com/kubernetes/enhancements/issues/592) - graduated to beta

# Release notes

1.21リリースの詳細は[リリースノート](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.21.md)から確認できます。

# Availability of release

Kubernetes 1.21は[GitHubからダウンロード](https://github.com/kubernetes/kubernetes/releases/tag/v1.21.0)可能です。また、Kubernetesを始めるための素晴らしいリソースがあります。[インタラクティブなチュートリアル](https://kubernetes.io/docs/tutorials/)を受けることも出来ますし、[kind](https://kind.sigs.k8s.io/)で、Dockerコンテナを使ってマシンにローカルクラスタを動かすことも出来ます。もし１からクラスターを作りたい場合はKelsey Hightowerによる[Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)チュートリアルをやってみてください。

# Release Team

このリリースは献身的な個人のグループによって作り上げられました。世界中で起きていることのまっただ中でチーム一眼となっていました。リリースリードのNabarunPalと、リリースチームの他のすべての人がお互いをサポートし、コミュニティに1.21リリースを提供するために一生懸命働いてくれたことに心から感謝します。

# Release Logo

![](https://d33wubrfki0l68.cloudfront.net/a5d269732fd8619bf45c77613e6d9be2c42618a3/9eb18/images/blog/2021-04-08-kubernetes-release-1.21/globe_250px.png)

1.21のリリースロゴはリリースチームのグローバルな性質を描写しています。リリースチームメンバーはUTC+8からUTC-8のタイムゾーンに分散していました。リリースチームの多様性は多くのチャレンジがありましたが、非同期のコミュニケーションを多く取り入れてこれに取り組みました。七角形の地球はコミュニティが直面する課題を克服するという完全な決意を意味します。楽しいKuberntesのパックを作るための3ヶ月に渡るリリースチームの驚くべきチームワークを祝います。

ロゴは、インドを拠点とする独立したデザイナーの[Aravind Sekar](https://www.behance.net/noblebatman)によってデザインされました。Aravindは、PyConIndiaのようなオープンソースコミュニティの設計作業を支援します。

# User Highlights
- CNCFは新たに世界の47の組織がメンバーとなることを歓迎します。2021年の初めにクラウドネイティブ技術の発展させましょう。これらの[メンバー](https://www.cncf.io/announcements/2021/02/24/cloud-native-computing-foundation-welcomes-47-new-members-at-the-start-of-2021/)は2021/5/4 - 7 の[KubeCon + CloudNativeCom EU – Virtual](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/) と2021/10/12 - 15 の K[ubeCon + CloudNativeCon NA in Los Angeles](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/)を含むCNCFのKubeCon + CloudNativeCon 2021に参加予定です。

:pen: 2021/10 のKubeConはVirtualイベントではないようです。

# Project Velocity

[CNCF K8s DevStats project](https://k8s.devstats.cncf.io/)はKubernetesやそのサブプロジェクトのVelocityに関わる様々な観点のデータを集約しています。ここでは個人の貢献に関わるものや貢献している企業の数値、このエコシステムを進化させる努力の深さや幅に関するきちんとしたイラストも含んでいます。
12週間(1月11日〜4月8日)だったv1.21リリースサイクルでは、[999の企業](https://k8s.devstats.cncf.io/d/9/companies-table?orgId=1&var-period_name=v1.20.0%20-%20now&var-metric=contributions)と[1279の個人](https://k8s.devstats.cncf.io/d/66/developer-activity-counts-by-companies?orgId=1&var-period_name=v1.20.0%20-%20now&var-metric=contributions&var-repogroup_name=Kubernetes&var-country_name=All&var-companies=All)による貢献が成されました。

# Ecosystem Updates
- 世界的なアジアのコミュニティ対する人種差別と攻撃の高まりを受け、[CNCFブログ](https://www.cncf.io/blog/2021/03/18/statement-from-cncf-general-manager-priyanka-sharma-on-the-unacceptable-attacks-against-aapi-and-asian-communities/)のCNCFGeneral Priyanka Sharmaの声明を読んでください。そこでは包括的な価値と多様性に基づく回復力(resilience)に関するコミュニティの取り組みの復活について書かれています。
- 現在我々はデフォルトブランチの名前を master から main へ移行中です。詳細は[こちら](https://kubernetes.io/blog/2021/04/08/kubernetes-1-21-release-announcement/k8s.dev/rename)のガイドラインを読んでください。
- CNCFとLinux Foundationは新しいトレーニングコースである[LFS260 – Kubernetes Security Essentials](https://training.linuxfoundation.org/training/kubernetes-security-essentials-lfs260/)の開始をアナウンスしました。Kubernetesプラットフォームとコンテナベースアプリケーションのセキュリティに関する広範囲なベストプラクティスに関連する技術と知識を提供することに加えて、このコースは最近始まった[Certified Kubernetes Security Specialist ](https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist/)試験の準備としても良い内容となっています。

# Event Updates
- KubeCon + CloudNativeCon Europe 2021 が2021/5/4 - 7 で開催されます。カンファレンスに関するより多くの情報は[こちら](https://events.linuxfoundation.org/kubecon-cloudnativecon-europe/)から参照してください。
- Kubernetes Community Days が開催予定です。2021 2Qにはアフリカとバンガロールで始まります。

# Upcoming release webinar
2021/5/13に開催のWebinarでKubernetes 1.21のリリースチームのメンバーと会って、このリリースの主要機能であるIPv4/IPv6デュアルスタックサポート、PersistentVolume Health Monitor、Immutable Secrets and ConfigMapsやその他多くを知ってください。こちらから登録：https://community.cncf.io/events/details/cncf-cncf-online-programs-presents-cncf-live-webinar-kubernetes-121-release/

# Get Involved
もしKubernetesコミュニティへの貢献について興味がある場合は、Special Interest Groups(SIGs)はスタートとして良いでしょう。あなたの興味と合う人が多くいます!あなたがコミュニティと共有したい物があれば、週次のコミュニティミーティングに参加したり、次のチャンネルを使ってみてください。
- Kubernetesへの貢献に関するより多くの情報は[Kubernetes Contributor website](https://www.kubernetes.dev/)で見つけてください。
- 最新のアップデートはツイッターの[@Kubernetesio](https://twitter.com/kubernetesio)をフォローしてください。
- コミュニティでのディスカッションについては[Discuss](https://discuss.kubernetes.io/)に参加してください。
- [Slack](http://slack.k8s.io/)でコミュニティに参加してください。
- あなたのKubernetesの[ストーリー](https://github.com/cncf/foundation/blob/master/case-study-guidelines.md)を共有してください。
- より多くのKubernetesに関わる出来事は[ブログ](https://kubernetes.io/blog/)を読んでください。
- [Kubernetes Release Team](https://github.com/kubernetes/sig-release/tree/master/release-team)についてより知ってください。

© 2021 The Kubernetes Authors | Documentation Distributed under CC BY 4.0

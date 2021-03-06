<properties
	pageTitle="HDInsight での R とは何ですか? HDInsight の R Server (プレビュー) の概要 | Microsoft Azure"
	description="HDInsight の R Server (プレビュー) の概要と、R Server を使用してビッグ データ分析用のアプリケーションを作成する方法について説明します。"
	services="hdinsight"
	documentationCenter=""
	authors="jeffstokes72"
	manager="paulettm"
	editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="08/17/2016"
   ms.author="jeffstok"/>


# HDInsight の R Server (プレビュー) の概要

Microsoft Azure HDInsight Premium では、Azure で HDInsight クラスターを作成する際に、オプションとして Microsoft R Server を使用できるようになりました。この新しい機能により、データ サイエンティスト、統計学者、R プログラマは、HDInsight でのスケーラブルで分散型の分析手法にオンデマンドでアクセスできるようになります。

クラスターは、現在のプロジェクトやタスクに合わせてサイズを設定し、不要になったら破棄できます。クラスターは Azure HDInsight の一部であるため、エンタープライズ レベルの 24 時間 365 日のサポートと稼働時間 99.9% の SLA が提供され、Azure エコシステムの他のコンポーネントとも柔軟に統合できます。

>[AZURE.NOTE] R Server は HDInsight Premium でのみ使用できます。現在のところ、HDInsight Premium は Hadoop クラスターと Spark クラスターでのみ使用できます。そのため、R Server は現在、HDInsight の Hadoop クラスターと Spark クラスターのみで使用できます。詳細については、[HDInsight で利用可能なさまざまなサービス レベルと Hadoop コンポーネント](hdinsight-component-versioning.md)に関する記事をご覧ください。

HDInsight の R Server は、Azure BLOB ストレージに読み込まれた大規模なデータセットで R ベースの分析を行うための最新の機能を備えています。R Server はオープン ソースの R を基盤としているため、自社で構築する R ベースのアプリケーションでは、8000 以上のオープン ソース R パッケージと、R Server に付属する Microsoft のビッグ データ分析パッケージである ScaleR のルーチンを活用できます。

Premium クラスターのエッジ ノードは、クラスターへの接続と R スクリプトの実行に便利な場所です。エッジ ノードでは、エッジ ノード サーバーのコア間で、ScaleR の並列化された分散関数を実行できます。また、ScaleR の Hadoop Map Reduce または Spark コンピューティング コンテキストを使用して、クラスターのノード間でこれらの関数を実行することもできます。

分析によって得られたモデルや予測は、ダウンロードしてオンプレミスで使用できます。また、[Azure Machine Learning Studio](http://studio.azureml.net) [Web サービス](../machine-learning/machine-learning-publish-a-machine-learning-web-service.md)などを使用して、Azure の別の場所で運用化することもできます。

## HDInsight での R の使用

HDInsight クラスターに R Server を組み込むには、Azure ポータルを使用してクラスターを作成する際に、Premium オプションを使用して Hadoop クラスターまたは Spark クラスターを作成する必要があります。どちらのクラスターの種類でも、クラスターのデータ ノード上の R Server と、R Server ベースの分析のランディング ゾーンとしてのエッジ ノードが含まれた同じ構成を使用します。クラスター作成の詳細な手順については、[HDInsight での R Server の使用](hdinsight-hadoop-r-server-get-started.md)に関する記事をご覧ください。

## データ ストレージ オプションについて

HDInsight クラスターの既定のストレージは、HDFS ファイル システムが BLOB コンテナーにマップされている BLOB Storage です。これにより、クラスター ストレージにアップロードされるデータや分析中にクラスター ストレージに書き込まれるデータがどのようなものであっても、確実に永続化されます。[AzCopy](../storage/storage-use-azcopy.md) ユーティリティを使用すると、BLOB との間でデータをコピーできます。

Blob Storage を使用するだけでなく、クラスターで [Azure Data Lake Storage](https://azure.microsoft.com/services/data-lake-store/) を使用することもできます。Data Lake を使用すると、BLOB Storage と HDFS 用 Data Lake ストレージの両方を使用できます。

エッジ ノードで使用するストレージ オプションとして、[Azure Files](../storage/storage-how-to-use-files-linux.md) を使用することもできます。Azure Files では、Azure Storage で作成されたファイル共有を Linux ファイル システムにマウントできます。HDInsight クラスターの R Server 用データ ストレージ オプションの詳細については、[HDInsight クラスターの R Server 用ストレージ オプション](hdinsight-hadoop-r-server-storage.md)に関する記事をご覧ください。

## クラスターでの R Server へのアクセス

R Server を含むクラスターを作成したら、SSH/PuTTY を使用してクラスターのエッジ ノードの R コンソールに接続できます。また、オプションの RStudio Server IDE をエッジ ノードにインストールすると、ブラウザーを介して接続することもできます。RStudio Server のインストールの詳細については、[HDInsight クラスターへの RStudio Server のインストール](hdinsight-hadoop-r-server-install-r-studio.md)に関する記事をご覧ください。

## R スクリプトの開発と実行

ユーザーが作成して実行する R スクリプトでは、ScaleR ライブラリの並列化された分散ルーチンに加え、8000 以上のオープン ソース R パッケージを使用できます。一般に、エッジ ノードの R Server で実行されるスクリプトは、そのノードの R インタープリター内で実行されます。コンピューティング コンテキストが Hadoop Map Reduce (RxHadoopMR) または Spark (RxSpark) に設定された ScaleR 関数を呼び出すステップは例外です。

この場合、関数は参照先データに関連付けられたクラスターのデータ (タスク) ノード間に分散した形で実行されます。各種コンピューティング コンテキスト オプションの詳細については、[HDInsight Premium の R Server のコンピューティング コンテキスト オプション](hdinsight-hadoop-r-server-compute-contexts.md)に関する記事をご覧ください。

## モデルの運用化

データ モデリングが完了したら、モデルを運用化して、Azure とオンプレミスの両方で新しいデータについての予測を行うことができます。このプロセスはスコア付けと呼ばれます。次に例をいくつか示します。

### HDInsight でのスコア付け

HDInsight でスコア付けを行うには、モデルを呼び出して、ストレージ アカウントに読み込んだ新しいデータ ファイルについて予測を行う R 関数を作成します。その後、予測をストレージ アカウントに保存します。このルーチンは、クラスターのエッジ ノードでオンデマンドで実行することも、スケジュールされたジョブを使用して実行することもできます。

### Azure Machine Learning でのスコア付け

Azure Machine Learning Web サービスを使用してスコア付けを行うには、[AzureML](https://cran.r-project.org/web/packages/AzureML/vignettes/getting_started.html) と呼ばれるオープン ソースの Azure Machine Learning R パッケージを使用して、モデルを Azure Web サービスとして発行します。便宜上、このパッケージはエッジ ノードにあらかじめインストールされています。次に、Machine Learning の機能を使用して Web サービスのユーザー インターフェイスを作成し、スコア付けが必要なときに Web サービスを呼び出します。

この方法を選択した場合、ScaleR モデル オブジェクトを、Web サービスで使用する同等のオープン ソース モデル オブジェクトに変換する必要があります。この変換は、ScaleR 強制型変換関数 (アンサンブルベースのモデルの `as.randomForest()` など) を使用して実行できます。


### オンプレミスのスコア付け

モデルの作成後にオンプレミスのスコア付けを行うには、R でモデルをシリアル化し、モデルをダウンロードして逆シリアル化してから、新しいデータのスコア付けに使用します。新しいデータのスコア付けは、「[HDInsight でのスコア付け](#scoring-in-hdinsight)」で前述した方法を使用するか、[DeployR](https://deployr.revolutionanalytics.com/) を使用して実行できます。

## クラスターの管理

### R パッケージのインストールと管理

使用する R パッケージのほとんどは、R スクリプトの大半が実行されるエッジ ノードに配置する必要があります。エッジ ノードに追加の R パッケージをインストールするには、R の通常の `install.packages()` メソッドを使用します。

ほとんどの場合、クラスターで ScaleR ライブラリのルーチンを使用するだけであれば、データ ノードに追加の R パッケージをインストールする必要はありません。ただし、データ ノードで **rxExec** または **RxDataStep** 実行の使用をサポートするために、追加のパッケージが必要になる場合があります。

このような場合、クラスターの作成後にスクリプト アクションを使用して追加のパッケージを指定する必要があります。詳細については、[R Server を含む HDInsight クラスターの作成](hdinsight-hadoop-r-server-get-started.md)に関する記事をご覧ください。

### Hadoop Map Reduce のメモリ設定の変更

クラスターに変更を加えることで、Map Reduce ジョブの実行時に R Server で使用可能なメモリ容量を変更できます。クラスターを変更するには、クラスターの Azure ポータル ブレードで使用できる Apache Ambari UI を使用します。クラスターの Ambari UI にアクセスする手順については、「[Ambari Web UI を使用した HDInsight クラスターの管理](hdinsight-hadoop-manage-ambari.md)」をご覧ください。

次のように、**RxHadoopMR** の呼び出しで Hadoop スイッチを使用して、R Server で使用可能なメモリ容量を変更することもできます。

	hadoopSwitches = "-libjars /etc/hadoop/conf -Dmapred.job.map.memory.mb=6656"  

### クラスターのスケーリング

既存のクラスターは、ポータルを使用してスケールアップまたはスケールダウンできます。スケーリングにより、大規模な処理タスクに必要な追加容量が得られます。また、クラスターがアイドル状態のときに、クラスターを縮小することもできます。クラスターのスケーリングの手順については、[HDInsight クラスターの管理](hdinsight-administer-use-portal-linux.md)に関する記事をご覧ください。

### システムのメンテナンス

OS 修正プログラムや他の更新プログラムを適用するために、HDInsight クラスターの基になる Linux VM で時間外にメンテナンスが実施されます。通常、メンテナンスは毎週月曜日と木曜日の午前 3 時 30 分 (VM のローカル時刻に基づく) に実施されます。更新は、4 分の 1 以上のクラスターに同時に影響を与えないような方法で実行されます。

ヘッド ノードが冗長化されており、すべてのデータ ノードが影響を受けるわけではないため、この期間に実行中のジョブは速度が低下する場合がありますが、完了するまで引き続き実行されます。クラスターの再構築を必要とする致命的な障害が発生しない限り、カスタム ソフトウェアやローカル データはメンテナンス イベントで保持されます。

## HDInsight クラスターの R Server の IDE オプションについて

HDInsight Premium クラスターの Linux エッジ ノードは、R ベースの分析のランディング ゾーンです。クラスターへの接続後、Linux コマンド プロンプトで「**R**」と入力して、R Server へのコンソール インターフェイスを起動できます。別のウィンドウで R スクリプト開発用のテキスト エディターを実行し、必要に応じてスクリプトのセクションを切り取って R コンソールに貼り付けると、コンソール インターフェイスの操作を改善できます。

R スクリプト開発用のより高度なツールとして、Microsoft が最近発表した [R Tools for Visual Studio](https://www.visualstudio.com/ja-JP/features/rtvs-vs.aspx) (RTVS) など、デスクトップで使用する R ベースの IDE があります。これは、[RStudio](https://www.rstudio.com/products/rstudio-server/) のデスクトップおよびサーバー ツール ファミリーです。また、Walware の Eclipse ベースの [StatET](http://www.walware.de/goto/statet) を使用することもできます。

Linux のエッジ ノード自体に IDE をインストールすることもできます。よく使用されている [RStudio Server](https://www.rstudio.com/products/rstudio-server/) は、リモート クライアント向けにブラウザー ベースの IDE を提供します。HDInsight Premium クラスターのエッジ ノードに RStudio Server をインストールすると、クラスターの R Server を使用した R スクリプトの開発と実行に IDE の機能をフルに活用できるようになり、R コンソールよりも生産性を大幅に向上させることができます。RStudio Server を使用する場合は、[HDInsight クラスターへの RStudio Server のインストール](hdinsight-hadoop-r-server-install-r-studio.md)に関する記事をご覧ください。

## 価格について

R Server を含む HDInsight Premium クラスターに関する料金は、Standard HDInsight クラスターの料金と同様に体系化されています。料金は、ネーム、データ、エッジの各ノードの基になる VM のサイズに基づきますが、Premium 向けのコア時間の割増分が加算されます。パブリック プレビュー中の価格や 30 日間無料試用版の提供状況など、HDInsight Premium の価格の詳細については、「[HDInsight の価格](https://azure.microsoft.com/pricing/details/hdinsight/)」をご覧ください。

## 次のステップ

HDInsight クラスターで R Server を使用する方法の詳細については、以下のリンクをご覧ください。

- [HDInsight での R Server の使用](hdinsight-hadoop-r-server-get-started.md)

- [HDInsight Premium への RStudio Server の追加に関する記事](hdinsight-hadoop-r-server-install-r-studio.md)

- [HDInsight の R Server (プレビュー) の計算コンテキストのオプション](hdinsight-hadoop-r-server-compute-contexts.md)

- [Azure Storage options for R Server on HDInsight Premium (HDInsight Premium での R Server の Azure Storage オプション)](hdinsight-hadoop-r-server-storage.md)

<!---HONumber=AcomDC_0817_2016-->
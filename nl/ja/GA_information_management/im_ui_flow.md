---

copyright:
years: 2017
lastupdated: "2017-08-10"

---

{:new_window: target="\_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}

# Web インターフェースを使用したデータ管理の概説 (ベータ)
{: #gs_web}

**重要:** Web インターフェースの {{site.data.keyword.iot_full}} データ管理フィーチャーは、限定されたベータ・プログラムの一部としてのみ使用できます。今後の更新によって、この機能の現行バージョンと互換性のない変更が行われる可能性があります。この機能を試して、[ご意見をお寄せください ![外部リンク・アイコン](../../../icons/launch-glyph.svg)](https://developer.ibm.com/answers/smart-spaces/17/internet-of-things.html){: new_window}。

{{site.data.keyword.iot_short_notm}} には、データ管理フィーチャーの一部としてデータを正規化して変換するために役立つオンライン・ツールが用意されています。提供されている [Watson IoT Platform データ管理 API ![外部リンク・アイコン](../../../icons/launch-glyph.svg "外部リンク・アイコン")](https://docs.internetofthings.ibmcloud.com/apis/swagger/v0002/state-mgmt.html){:new_window} に加えて、簡易インターフェース作成機能または拡張インターフェース作成機能を使用して、リソースを構成し、デバイス・データのマッピングを開始することができます。オンラインのインターフェース作成機能のガイドを活用して、ユーザー・インターフェースの各手順を進めることができます。

データ管理フィーチャーとその利点については、[データ管理の概要](../GA_information_management/ga_im_device_twin.html#device_twins)を参照してください。

構成する必要があるリソースについては、[データ管理の解説](../GA_information_management/ga_im_definitions.html#definitions_resource)を参照してください。

簡易インターフェース作成機能を使用して 1 つの物理インターフェースを追加すると、1 つの論理インターフェースが自動的に作成され、これら 2 つがマップされます。拡張インターフェース作成機能を使用すると、構成をより細かく制御できます。1 つの物理インターフェースと 1 つ以上の論理インターフェースを追加した後、それらのインターフェース間のマッピングを追加します。

これらのインターフェースとマッピングは、ドラフト形式で追加されます。これらを追加して検証したら、アクティブにする必要があります。ドラフト・リソースとアクティブ・リソースについて詳しくは、[リソースの作成、更新、アクティブ化、非アクティブ化](../GA_information_management/ga_im_definitions.html#draft_active_resources)を参照してください。



## 大まかな流れ
{: #interface_flow}

以下の手順を使用して、データ管理フィーチャーを使用して開始する必要があるリソースを構成します。

**始めに**
この手順では、接続された登録済みのデバイスが少なくとも 1 つあるデバイス・タイプが存在していることを前提としています。これらの前提条件や、デバイス・タイプの作成方法とデバイスの登録方法の手順について詳しくは、[データ管理フィーチャーを使用するためのデバイスの構成 (ベータ)](im_config_devices.html) を参照してください。

1. データを正規化して変換する対象となるデバイス・タイプを選択します。
2. 「簡易インターフェース作成機能」または「拡張インターフェース作成機能」のいずれかを選択します。

a. シンプルなフロー  
   i. [ドラフト物理インターフェースを作成します](#create_physical_interface_simple)。
  
   
b. 拡張フロー (Advanced flow)  
   i. [ドラフト物理インターフェースを作成します](#create_physical_interface_advanced)。
  
   ii. [1 つ以上のドラフト論理インターフェースを作成します](#create_logical_interface)。(シンプルなフローを使用した場合、このステップは自動的に完了します。)  
   iii. [物理インターフェースを各論理インターフェースにマップします](#create_interface_mappings)。  
     
     
3. [通知設定を行います](#set_notifications)。
4. [構成をアクティブにします](#validate_activate)。

*注:* データ管理フィーチャーは API を使用して構成することもできます。API を使用してデバイス・データの正規化と変換を行うようにデータ管理フィーチャーを構成する方法についての詳細は、[データ管理の概説]{ga_im_example.html#im_example} を参照してください。[ステップバイステップ・ガイド: 共通インターフェースによってデバイスを処理する方法を詳細に示す例](../GA_information_management/ga_im_index_scenario.html#scenario)では、種類の異なる複数の温度計デバイスに対応したデバイス・タイプ論理インターフェースを作成するための一連の手順について説明しています。

API についての詳細は、[{{site.data.keyword.iot_short_notm}} HTTP REST API ![外部リンク・アイコン](../../../icons/launch-glyph.svg "外部リンク・アイコン")](https://docs.internetofthings.ibmcloud.com/apis/swagger/v0002/state-mgmt.html){:new_window} の資料を参照してください。


## ドラフト物理インターフェースの作成 (シンプルなフロー)
{: #create_physical_interface_simple}

1. メイン・ナビゲーション・メニューで**「デバイス」**を選択します。
2. **「デバイス・タイプ」**を選択し、インターフェースを作成するデバイスのタイプを選択します。[新規デバイス・タイプを作成する] こともできます。
3. 必要に応じて、デバイス・タイプの情報を表示して更新し、変更内容を保存します。
4. **「インターフェース」**をクリックします。
5. **「シンプルなフロー」**をクリックして、簡易インターフェース作成機能を開きます。
6. **「インターフェースの作成」**をクリックします。
7. **「プロパティーの追加」**をクリックして、物理インターフェースへのイベントとプロパティーの追加を開始します。  
 a. **「インターフェースにプロパティーを追加」**ウィンドウで、インターフェースに追加する各イベントを選択します。選択したデバイス・タイプの接続済みデバイスのアクティブ・イベントをシステムが listen します。最後にキャッシュされたイベントのデバイス ID を選択することも、リストから別のイベントを選択することもできます。  
  b. イベントからプロパティーを選択して、**「追加」**をクリックします。  
  c. 必要に応じて、プロパティーをさらに追加します。
8. **「完了」**をクリックします。物理インターフェースが作成されます。さらに詳細を追加する場合、**「拡張インターフェース作成機能の使用」**リンクを選択して、インターフェースをアップグレードすることができます。


## ドラフト物理インターフェースの作成 (拡張フロー)
{: #create_physical_interface_advanced}

1. メイン・ナビゲーション・メニューで**「デバイス」**を選択します。
2. **「デバイス・タイプ」**を選択し、インターフェースを作成するデバイスのタイプを選択します。[新規デバイス・タイプを作成する] こともできます。
2. デバイス・タイプの情報を表示して、**「インターフェース」**をクリックします。
4. **「拡張フロー (Advanced Flow)」**をクリックして、拡張インターフェース作成機能を開きます。
5. **「インターフェースの作成」**をクリックします
7. **「物理インターフェースの作成」**ページの**「ID」**タブで、物理インターフェースの名前と説明を入力します。
7. オプション: インターフェースでエッジ対応デバイスを使用している場合は、エッジ分析スイッチをオンにします。
8. **「次へ」**をクリックします。
9. イベント・タイプとペイロードを追加することによって、物理インターフェースを定義します。
 a. **「イベント・タイプの作成 (Create Event Type)」**をクリックします。 
 b. 最後にキャッシュされたイベントを選択するか、リストからイベントを選択するか、または手動でイベントを追加します。
10. **「完了」**をクリックします。物理インターフェースがドラフト形式で作成されます。続いて、1 つ以上の論理インターフェースを追加することができます。

## ドラフト論理インターフェースの作成
{: #create_logical_interface}

1. ドラフト物理インターフェースを作成したら、**「論理インターフェースの追加 (Add Logical Interface)」**をクリックします。
2. **「論理インターフェースの作成 (Create Logical Interface)」**ページの**「ID」**タブで、論理インターフェースの名前と説明を入力します。
3. **「次へ」**をクリックします。
4. **「プロパティーの追加」**をクリックして、論理インターフェースのプロパティーの定義を開始します。
5. **「プロパティーの作成 (Create Property)」**ウィンドウで、論理インターフェースに追加する物理インターフェースからプロパティーを選択した後、その変更内容を保存します。
6. 必要に応じて論理インターフェースをさらに追加してから、**「次へ」**をクリックします。
7. {: #set_notifications}インターフェース用の通知設定をセットアップします。次のオプションのいずれかを選択して、デバイス状態の通知を送信するタイミングを決定します。
 - イベント通知機能なし (No Event Notifiers) - 通知は送信されません。REST API を使用して、デバイス状態の変化の情報を取得できます。
 - 状態の変化時 (On State Change) - デバイス状態が変化したときに通知されます。
 - イベントごと (On Every Event) - プラットフォームで対象のデバイスのイベントを処理するたびに (そのイベントで状態が変化しない場合でも) 通知されます。
8. **「完了」**をクリックします。各インターフェースがドラフト形式で作成されます。
9. {: #validate_activate} **「デプロイ」**をクリックして、構成を検証します。
10. **「デプロイ」**をクリックして、構成をアクティブにします。構成状況がデプロイ済みに設定されます。 

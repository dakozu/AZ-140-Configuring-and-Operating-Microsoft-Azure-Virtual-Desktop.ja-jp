---
lab:
  title: 'ラボ: ホスト プールに自動スケーリングを実装する (AD DS)'
  module: 'Module 5: Monitor and Maintain an AVD Infrastructure'
---

# ラボ - ホスト プールに自動スケールを実装する (AD DS)
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプションの所有者または共同作成者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントと、Azure サブスクリプションに関連付けられた Microsoft Entra テナントのグローバル管理者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウント。
- 完了したラボ: **Azure Virtual Desktop (AD DS) のデプロイを準備する**
- 完了したラボ **Azure portal を使用してホスト プールとセッション ホストをデプロイする (AD DS)**

## 推定所要時間

60 分

## ラボのシナリオ

Active Directory Domain Services (AD DS) 環境で Azure Virtual Desktop セッション ホストの自動スケーリングを構成する必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- Azure Virtual Desktop セッション ホストの自動スケーリングを構成する
- Azure Virtual Desktop セッション ホストの自動スケーリングを検証する

## ラボ ファイル

- なし

## 手順

### 演習 1: Azure Virtual Desktop セッション ホストの自動スケーリングを構成する

この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop セッション ホストのスケーリングの準備をする
2. Azure Virtual Desktop の自動スケーリングを追跡するための診断を設定する
3. Azure Virtual Desktop セッション ホストのスケーリング プランを作成する

#### タスク 1: Azure Virtual Desktop セッション ホストのスケーリングの準備をする

1. ラボ コンピューターから Web ブラウザーを起動して [Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を入力してサインインします。
1. ラボ コンピューターの Azure portal を表示しているブラウザー ウィンドウで、[**Cloud Shell**] ウィンドウの **PowerShell** セッションを開きます。

   >**注**: 自動スケールで使用する予定のホスト プールは、**MaxSessionLimit** パラメーターの既定値以外で構成する必要があります。 この値は、Azure portal のホスト プール設定で構成するか、この例のように **Update-AzWvdHostPool** の Azure PowerShell コマンドレットを実行して構成します。 Azure portal でプールを作成するとき、または **New-AzWvdHostPool** Azure PowerShell コマンドレットを実行するときに、明示的に構成することもできます。

1. [Cloud Shell] ウィンドウの PowerShell セッションで次のコマンドを実行して、**az140-21-hp1** ホスト プールの **MaxSessionLimit** パラメーターの値を **2** に設定します。 

   ```powershell
   Update-AzWvdHostPool -ResourceGroupName 'az140-21-RG' `
   -Name az140-21-hp1 `
   -MaxSessionLimit 2
   ```

   >**注**: このラボでは、自動スケーリング動作のトリガーを容易にするために、**MaxSessionLimit** パラメーターの値が人為的に低く設定されています。

   >**注**: 最初のスケーリング プランを作成する前に、Azure サブスクリプションをターゲット スコープとして、**Desktop Virtualization Power On Off 共同作成者** RBAC ロールを Azure Virtual Desktop に割り当てる必要があります。 

1. Azure portal が表示されているブラウザー ウィンドウで、[Cloud Shell] ウィンドウを閉じます。
1. Azure portal で**サブスクリプション**を検索して選択し、サブスクリプションの一覧から Azure Virtual Desktop リソースを含むサブスクリプションを選択します。 
1. [サブスクリプション] ページで、[**アクセス制御 (IAM)**] を選択します。
1. [**アクセス制御 (IAM)**] ページのツールバーで **[+ 追加] ボタン**を選択して、ドロップダウン メニューで [**ロールの割り当ての追加**] を選択します。
1. **[ロール割り当ての追加]** ブレードの **[ロール]** タブで、次の設定を指定して、**[次へ]** を選択します。

   |設定|Value|
   |---|---|
   |職務権限ロール|**デスクトップ仮想化電源オン/オフ共同作成者**|

1. **[ロール割り当ての追加]** ブレードの **[メンバー]** タブで、**[+ メンバーの選択]** をクリックし、次の設定を指定して、**[選択]** をクリックします。 

   |設定|Value|
   |---|---|
   |選択|**Azure Virtual Desktop** または **Windows Virtual Desktop**|

1. **[ロール割り当ての追加]** ブレードで、**[レビューと割り当て]** を選択します

   >**注**: どちらの値を使用するかは、**Microsoft.DesktopVirtualization** リソース プロバイダーが Azure テナントに最初に登録された時期によって異なります。

1. **[確認と割り当て]** タブで、 **[確認と割り当て]** を選択します。

#### タスク 2:Azure Virtual Desktop の自動スケーリングを追跡するように診断を設定する

1. ラボ コンピューターの Azure portal を表示しているブラウザー ウィンドウで、[**Cloud Shell**] ウィンドウの **PowerShell** セッションを開きます。

   >**注**: 自動スケーリング イベントを格納するには、Azure ストレージ アカウントを使用します。 このタスクに示すように、Azure portal から直接作成することも、Azure PowerShell を使用することもできます。

1. [Cloud Shell] ウィンドウの PowerShell セッションで次のコマンドを実行して、新しい Azure Storage アカウントを作成します。

   ```powershell
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName 'az140-11-RG').Location
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   $suffix = Get-Random
   $storageAccountName = "az140st51$suffix"
   New-AzStorageAccount -Location $location -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName Standard_LRS
   ```

   >**注**: ストレージ アカウントがプロビジョニングされるまで待ちます。

1. Azure portal が表示されているブラウザー ウィンドウで、[Cloud Shell] ウィンドウを閉じます。
1. ラボ コンピューターの Azure portal が表示されているブラウザーで、**az140-21-hp1** ホスト プールのページに移動します。
1. **[az140-21-hp1]** ページで、**[診断設定]** を選択したら、**[+ 診断設定の追加]** を選択します。
1. **[診断設定]** ページの **[診断設定名]** テキストボックスに「**az140-51-scaling-plan-diagnostics**」と入力し、**[カテゴリ グループ]** セクションで **[プールされたホスト プールの自動スケーリング ログ]** を選択します。 
1. 同じページの [**宛先の詳細**] セクションで、[**ストレージ アカウントへのアーカイブ**] を選択し、[**ストレージ アカウント**] ドロップダウン リストで、**az140st51** プレフィックスで始まるストレージ アカウント名を選択します。
1. **[保存]** を選択します。

#### タスク 3:Azure Virtual Desktop セッション ホストのスケーリング プランを作成する

1. ラボ コンピューターの Azure portal が表示されているブラウザーで、**Azure Virtual Desktop** を検索して選択します。 
1. [**Azure Virtual Desktop**] ページで、[**スケーリング プラン**] を選択し、[**+ 作成**] を選択します。
1. **[スケーリング プランの作成]** ウィザードの **[基本]** タブで、次の情報を指定して **[次へ:スケジュール >]** を選択します (他は既定値のままにします)。

   |設定|値|
   |---|---|
   |リソース グループ|**az140-51-RG**|
   |名前|**az140-51-scaling-plan**|
   |場所|前のラボでセッション ホストをデプロイしたのと同じ Azure リージョン|
   |フレンドリ名|**az140-51 scaling plan**|
   |タイム ゾーン|ローカル タイム ゾーン|

   >**注**: 除外タグを使用すると、スケーリング操作から除外するセッション ホストのタグ名を指定できます。 たとえば、例外タグ excludeFromScaling を使用することで、メンテナンス中に自動スケーリングがドレイン モードをオーバーライドしないようにするために、ドレイン モードに設定されている VM にタグを付けることができます。 

1. **スケーリング プランの作成**ウィザードの [**スケジュール**] タブで、[**+ スケジュールの追加**] を選択します。
1. **スケジュールの追加**ウィザードの [**全般**] タブで、次の情報を指定し、[**次へ**] をクリックします。

   |設定|Value|
   |---|---|
   |スケジュール名|**az140-51-schedule**|
   |繰り返し|**7 を選択** (すべての曜日を選択)|

1. **スケジュールの追加**ウィザードの [**ランプアップ**] タブで、次の情報を指定し、[**次へ**] をクリックします。

   |設定|Value|
   |---|---|
   |開始時刻 (24 時間システム)|現在の時刻から 9 時間を引いた時刻|
   |負荷分散アルゴリズム|**幅優先**|
   |ホストの最小割合 (%)|**20**|
   |容量のしきい値 (%)|**60**|

   >**注**: ここで選択した負荷分散設定によって、元のホスト プール設定で選択した設定がオーバーライドされます。

   >**注**: ホストの最小割合には、常にオンにしておきたいセッション ホストの割合を入力します。 入力した割合が整数でない場合は、最も近い整数に切り上げされます。 

   >**注**: 容量のしきい値には、スケーリング アクションの実行をトリガーする使用可能なホスト プール容量の割合を入力します。 たとえば、最大セッション制限が 20 であるホスト プール内の 2 つのセッション ホストが稼働している場合、使用可能なホスト プールの容量は 40 になります。 容量のしきい値が 75% に設定され、セッション ホストに 30 を超えるユーザー セッションがある場合、自動スケーリングは 3 番目のセッション ホストを稼働させます。 これにより、使用可能なホスト プールの容量は 40 から 60 に変更されます。

1. **スケジュールの追加**ウィザードの [**ピーク時間**] タブで、次の情報を指定し、[**次へ**] をクリックします。

   |設定|Value|
   |---|---|
   |開始時刻 (24 時間システム)|現在の時刻から 8 時間を引いた時刻|
   |負荷分散アルゴリズム|**深さ優先**|

   >**注**: 開始時刻は、ランプアップ フェーズの終了時刻を指定します。

   >**注**: このフェーズの容量しきい値は、ランプアップ容量しきい値によって決まります。

1. **スケジュールの追加**ウィザードの [**ランプダウン**] タブで、次の情報を指定し、[**次へ**] をクリックします。

   |設定|Value|
   |---|---|
   |開始時刻 (24 時間システム)|現在の時刻から 2 時間を引いた時刻|
   |負荷分散アルゴリズム|**深さ優先**|
   |ホストの最小割合 (%)|"**10**"|
   |容量のしきい値 (%)|**90**|
   |Force logoff users(ユーザーの強制ログオフ)|**はい**|
   |ユーザーをログアウトして VM をシャットダウンするまでの遅延時間 (分)|**30**|

   >**注**: **ユーザーの強制ログオフ**が有効になっている場合、自動スケーリングでは、ユーザー セッションの数が最も少ないセッション ホストがドレイン モードになり、アクティブなすべてのユーザー セッションにシャットダウンが迫っているという通知が送信され、指定された遅延時間が経過した後に強制的にサインアウトされます。 自動スケーリングは、すべてのユーザー セッションをサインアウトした後、VM の割り当てを解除します。 

   >**注**: ランプダウン中に強制サインアウトを有効にしていない場合、アクティブなセッションまたは切断されたセッションがないセッション ホストの割り当てが解除されます。

1. **スケジュールの追加**ウィザードの [**ピーク時間外**] タブで、次の情報を指定し、[**追加**] をクリックします。

   |設定|Value|
   |---|---|
   |開始時刻 (24 時間システム)|現在の時刻から 1 時間を引いた時刻|
   |負荷分散アルゴリズム|**深さ優先**|

   >**注**: このフェーズの容量しきい値は、ランプダウン容量のしきい値によって決まります。

1. **スケーリング プランの作成**ウィザードの [**スケジュール**] タブに戻り、[**次へ: ホスト プールの割り当て >**] を選択します。
1. [**ホスト プールの割り当て**] ページの [**ホスト プールの選択**] ドロップダウン リストで **az140-21-hp1** を選択し、[**自動スケールの有効化**] チェックボックスが有効になっていることを確認し、[**確認と作成**] を選択して、[**作成**] を選択します。


### 演習 2: Azure Virtual Desktop のセッション ホストの自動スケーリングを検証する

この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop セッション ホストの自動スケーリングを検証する


#### タスク 1:Azure Virtual Desktop セッションホストの自動スケールを確認する

1. ラボ コンピューターの Azure portal を表示しているブラウザー ウィンドウで、[**Cloud Shell**] ウィンドウの **PowerShell** セッションを開きます。
1. [Cloud Shell] ウィンドウの PowerShell セッションで、次のコマンドを実行して、このラボで使用する Azure Virtual Desktop セッション ホストの Azure VM を開始します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**注**: セッション ホストの Azure VM が実行されるまで待ちます。

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザー ウィンドウで、**az140-21-hp1** ホスト プールのページに移動します。
1. **az140-21-hp1** ページで、[**セッション ホスト**] を選択します。
1. 少なくとも 1 つのセッション ホストが**シャットダウン**の状態で一覧表示されるまで待ちます。

   > **注**: セッション ホストの状態を更新するには、ページの更新が必要になる場合があります。

   > **注**:すべてのセッション ホストが 15 分後も使用可能なままの状態である場合は、**[az140-51-scaling-plan]** ページに戻り、**[ホストの最小比率 (%)]** **[ランプダウン]** 設定の値を減らします。

   > **注**: 1 つ以上のセッション ホストの状態が変更されると、自動スケーリング ログを Azure Storage アカウントで使用できるようになります。 

1. Azure portal で**ストレージ アカウント**を検索して選択し、[**ストレージ アカウント**] ページで、この演習で前に作成したストレージ アカウント (**az140st51** プレフィックスで始まる名前) を表すエントリを選択します。
1. [ストレージ アカウント] ページで、[**コンテナー**] を選択します。
1. コンテナーの一覧で、**[insights-logs-autoscaleevaluationpooled]** を選択します。
1. **[insights-logs-autoscaleevaluationpooled]** ページで、コンテナーに格納されている JSON 形式の BLOB を表すエントリに到達するまで、フォルダー階層を移動します。
1. BLOB エントリを選択し、ページの右端にある省略記号アイコンを選択し、ドロップダウン メニューで [**ダウンロード**] を選択します。
1. ラボ コンピューターで、ダウンロードした BLOB を任意のテキスト エディターで開き、その内容を確認します。 自動スケーリング イベントへの参照を見つけることができるはずです。この場合は、「割り当て解除」と検索すると、簡単に識別できるようになります。

   >**注**: 自動スケール イベントへの参照を含むサンプル BLOB コンテンツを次に示します。

   ```json
   host_Ring    "R0"
   Level    4
   ActivityId   "00000000-0000-0000-0000-000000000000"
   time "2023-03-26T19:35:46.0074598Z"
   resourceId   "/SUBSCRIPTIONS/AAAAAAAE-0000-1111-2222-333333333333/RESOURCEGROUPS/AZ140-51-RG/PROVIDERS/MICROSOFT.DESKTOPVIRTUALIZATION/SCALINGPLANS/AZ140-51-SCALING-PLAN"
   operationName    "ScalingEvaluationSummary"
   category "AutoscaleEvaluationPooled"
   resultType   "Succeeded"
   level    "Informational"
   correlationId    "ddd3333d-90c2-478c-ac98-b026d29e24d5"
   properties   
   Message  "Active session hosts are at 0.00% capacity (0 sessions across 3 active session hosts). This is below the minimum capacity threshold of 90%. 2 session hosts can be drained and deallocated."
   HostPoolArmPath  "/subscriptions/aaaaaaaa-0000-1111-2222-333333333333/resourcegroups/az140-21-rg/providers/microsoft.desktopvirtualization/hostpools/az140-21-hp1"
   ScalingEvaluationStartTime   "2023-03-26T19:35:43.3593413Z"
   TotalSessionHostCount    "3"
   UnhealthySessionHostCount    "0"
   ExcludedSessionHostCount "0"
   ActiveSessionHostCount   "3"
   SessionCount "0"
   CurrentSessionOccupancyPercent   "0"
   CurrentActiveSessionHostsPercent "100"
   Config.ScheduleName  "az140-51-schedule"
   Config.SchedulePhase "OffPeak"
   Config.MaxSessionLimitPerSessionHost "2"
   Config.CapacityThresholdPercent  "90"
   Config.MinActiveSessionHostsPercent  "5"
   DesiredToScaleSessionHostCount   "-2"
   EligibleToScaleSessionHostCount  "1"
   ScalingReasonType    "DeallocateVMs_BelowMinSessionThreshold"
   BeganForceLogoffOnSessionHostCount   "0"
   BeganDeallocateVmCount   "1"
   BeganStartVmCount    "0"
   TurnedOffDrainModeCount  "0"
   TurnedOnDrainModeCount   "1"
   ```


### 演習 3: ラボでプロビジョニングされた Azure VM を停止して割り当てを解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングされた Azure VM を停止して割り当てを解除する

>**注**: この演習では、関連するコンピューティング料金を最小限に抑えるために、このラボで使用された Azure VM の割り当てを解除します。

#### タスク 1: ラボでプロビジョニングされた Azure VM の割り当てを解除する

1. ラボ コンピューターに切り替えて、Azure portal を表示している Web ブラウザー ウィンドウの [**Cloud Shell**] ウィンドウで **PowerShell** セッションを開きます。
1. [Cloud Shell] ウィンドウの PowerShell セッションから次のコマンドを実行して、このラボで使用されているすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. [Cloud Shell] ウィンドウの PowerShell セッションから次のコマンドを実行して、このラボで使用したすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**: このコマンドは非同期で実行されるため (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。

# AZ-140:Microsoft Azure Virtual Desktop の構成と操作

- **[ラボへのリンク (HTML 形式)](https://microsoftlearning.github.io/AZ-140-Configuring-and-Operating-Microsoft-Azure-Virtual-Desktop/)**
- **あなたは MCT ですか?** - [MCT 向けの GitHub ユーザー ガイド](https://microsoftlearning.github.io/MCT-User-Guide/)をご覧ください
- **ラボの手順を手動で作成する必要がありますか?** - 手順は、[MicrosoftLearning/Docker-Build](https://github.com/MicrosoftLearning/Docker-Build) リポジトリで確認できます

## ここでの内容

- このコースをサポートするには、コースで使用される Azure サービスを最新の状態に保つために、コース コンテンツを頻繁に更新する必要があります。  ラボの手順とラボ ファイルは GitHub で公開しています。これにより、コースの作成者と MCT 間でのオープンな作業が可能となり、Azure プラットフォームの変更に合わせてコンテンツを最新の状態に保つことができます。

- これにより、今まで経験したことがないようなコラボレーション感がラボに生まれると期待しています。Azure が変更され、ライブ配信中にあなたがそれを最初に見つけた場合は、ラボ ソースで機能強化を行ってください。  仲間の MCT を支援しましょう。

## リリースされた MOC のファイルに対してこれらのファイルを使用する方法

- 講師ハンドブックと PowerPoint は、引き続きコースのコンテンツを教えるための主要なソースになるでしょう。

- GitHub 上のこれらのファイルは学生ハンドブックと組み合わせて使用するように設計されています。ただし、中央リポジトリとして GitHub 内にあるので、MCT とコース作成者が最新のラボ ファイルの共有ソースを持っている可能性があります。

- 講師は、ラボを行うたびに、最新の Azure サービスに合わせて修正された個所がないか GitHub を確認し、最新のラボ用ファイルを取得してください。

## 受講者ハンドブックの変更については?

- 受講者ハンドブックは四半期ごとに見直し、必要があれば通常の MOC リリースの手順を通して更新します。

## 貢献するには?

- すべての MCT は、GitHub repro のコードまたはコンテンツに pull request を送信できます。Microsoft とコース作成者は、必要に応じてコンテンツとラボ コードの変更をトリアージして追加します。

- バグ、変更、改善、アイデアを送信できます。  新しい Azure 機能を先に見つけたら、  新しいデモを提出しましょう。

## メモ

**ラボは、使用する ID プロバイダーに応じて、2 つの別々のトラックで構成されます。**

- Active Directory ドメイン サービス (AD DS)。 このトラックは、以下のラボで構成されています。

   - LAB_01L01_Prepare_for_deployment_of_AVD_ADDS.md
   - LAB_02L01_Deploy_host_pools_and_session_hosts_with_the_Azure_portal_ADDS.md
   - LAB_02L02_Implement_and_manage_storage_for_AVD_ADDS.md
   - LAB_02L03_Deploy_host_pools_and_hosts_with_ARM_templates_ADDS.md
   - LAB_02L04_Deploy_and_manage_host_pools_and_hosts_with_PowerShell_ADDS.md
   - LAB_02L05_Create_and_manage_session_host_images_ADDS.md
   - LAB_03L01_Configure_Conditional_Access_policies_for_AVD_ADDS.md
   - LAB_04L01_Implement_and_manage_AVD_profiles_ADDS.md
   - LAB_04L02_Package_AVD_applications_ADDS.md
   - LAB_05L01_Implement_autoscaling_in_host_pools_ADDS.md

- Azure Active Directory Domain Services (Azure AD DS)。 このトラックは、以下のラボで構成されています。

   - LAB_01L01_Prepare_for_deployment_of_AVD_AADDS.md
   - LAB_02L01_Create_and_configure_host_pools_and_session_hosts_AADDS.md
   - LAB_02L02_Implement_and_manage_storage_for_AVD_AADDS.md
   - LAB_04L01_Implement_and_manage_AVD_profiles_AADDS.md

### コース資料

MCT とパートナーが、これらの資料にアクセスし、学生に個別に提供することを強く推奨します。  進行中のクラスの一部としてラボ ステップにアクセスできるように、学生に直接 GitHub を指示するには、学生がコースの一部として別の UI にもアクセスする必要がありますが、これは混乱の原因となります。 個別のラボの手順を受け取る理由を学生に説明すると、クラウドベースのインターフェイスとプラットフォームが常に変化しているという性質を強調できます。 GitHub 上のファイルにアクセスするための Microsoft Learning サポートと GitHub サイトのナビゲーションのサポートは、このコースを教える MCT のみに限定されています。

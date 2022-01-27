# sap-api-integrations-classification-reads
sap-api-integrations-classification-reads は、外部システム(特にエッジコンピューティング環境)をSAPと統合することを目的に、SAP API で クラスデータを取得するマイクロサービスです。        
sap-api-integrations-classification-reads には、サンプルのAPI Json フォーマットが含まれています。       
sap-api-integrations-classification-reads は、オンプレミス版である（＝クラウド版ではない）SAPS4HANA API の利用を前提としています。クラウド版APIを利用する場合は、ご注意ください。       
https://api.sap.com/api/OP_API_CLFN_CLASS_SRV/overview   

## 動作環境  
sap-api-integrations-classification-reads は、主にエッジコンピューティング環境における動作にフォーカスしています。    
使用する際は、事前に下記の通り エッジコンピューティングの動作環境（推奨/必須）を用意してください。  
・ エッジ Kubernetes （推奨）    
・ AION のリソース （推奨)    
・ OS: LinuxOS （必須）    
・ CPU: ARM/AMD/Intel（いずれか必須）　　

## クラウド環境での利用
sap-api-integrations-classification-reads は、外部システムがクラウド環境である場合にSAPと統合するときにおいても、利用可能なように設計されています。    

## 本レポジトリ が 対応する API サービス
sap-api-integrations-classification-reads が対応する APIサービス は、次のものです。  

* APIサービス概要説明 URL: https://api.sap.com/api/OP_API_CLFN_CLASS_SRV/overview    
* APIサービス名(=baseURL): API_CLFN_CLASS_SRV

## 本レポジトリ に 含まれる API名
sap-api-integrations-classification-reads には、次の API をコールするためのリソースが含まれています。  

* A_ClfnClassForKeyDate（クラス）※クラス関連データを取得するために、ToCharc、ToClassDescription、と合わせて利用されます。
* ToCharc（クラス - 特性）
* ToClassDescription（クラス - 説明）

## API への 値入力条件 の 初期値
sap-api-integrations-classification-reads において、API への値入力条件の初期値は、入力ファイルレイアウトの種別毎に、次の通りとなっています。  

### SDC レイアウト

* inoutSDC.Class.Class（クラス）

## SAP API Bussiness Hub の API の選択的コール

Latona および AION の SAP 関連リソースでは、Inputs フォルダ下の sample.json の accepter に取得したいデータの種別（＝APIの種別）を入力し、指定することができます。  
なお、同 accepter にAll(もしくは空白)の値を入力することで、全データ（＝全APIの種別）をまとめて取得することができます。  

* sample.jsonの記載例(1)  

accepter において 下記の例のように、データの種別（＝APIの種別）を指定します。  
ここでは、"Class" が指定されています。

```
	"api_schema": "A_ClfnClassForKeyDate",
	"accepter": ["Class"],
	"class_code": "R2R_CL_REL_CEKKO",
	"deleted": false
```
  
* 全データを取得する際のsample.jsonの記載例(2)  

全データを取得する場合、sample.json は以下のように記載します。  

```
	"api_schema": "A_ClfnClassForKeyDate",
	"accepter": ["All"],
	"class_code": "R2R_CL_REL_CEKKO",
	"deleted": false
```

## 指定されたデータ種別のコール

accepter における データ種別 の指定に基づいて SAP_API_Caller 内の caller.go で API がコールされます。  
caller.go の func() 毎 の 以下の箇所が、指定された API をコールするソースコードです。  

```
func (c *SAPAPICaller) AsyncGetClass(class string, accepter []string) {
	wg := &sync.WaitGroup{}
	wg.Add(len(accepter))
	for _, fn := range accepter {
		switch fn {
		case "Class":
			func() {
				c.Class(class)
				wg.Done()
			}()
		default:
			wg.Done()
		}
	}

	wg.Wait()
}
```

## Output  
本マイクロサービスでは、[golang-logging-library-for-sap](https://github.com/latonaio/golang-logging-library-for-sap) により、以下のようなデータがJSON形式で出力されます。  
以下の sample.json の例は、SAP の クラスデータ が取得された結果の JSON の例です。  
以下の項目のうち、"Delete_mc" ～ "to_ClassDescription" は、/SAP_API_Output_Formatter/type.go 内 の type Class {}による出力結果です。  
"cursor" ～ "time"は、golang-logging-library-for-sap による 定型フォーマットの出力結果です。  

```
{
	"cursor": "/Users/latona2/bitbucket/sap-api-integrations-classification-reads/SAP_API_Caller/caller.go#L53",
	"function": "sap-api-integrations-classification-reads/SAP_API_Caller.(*SAPAPICaller).Class",
	"level": "INFO",
	"message": [
		{
			"Delete_mc": true,
			"Update_mc": true,
			"to_ClassCharacteristic_oc": true,
			"to_ClassDescription_oc": true,
			"to_ClassKeyword_oc": true,
			"to_ClassText_oc": true,
			"ClassInternalID": "1",
			"ClassType": "032",
			"ClassTypeName": "Release Strategy",
			"Class": "R2R_CL_REL_CEKKO",
			"ClassStatus": "1",
			"ClassStatusName": "Released",
			"ClassGroup": "",
			"ClassGroupName": "",
			"ClassSearchAuthGrp": "",
			"ClassClassfctnAuthGrp": "",
			"ClassMaintAuthGrp": "",
			"CreationDate": "2016-06-20T09:00:00+09:00",
			"LastChangeDate": "2016-06-20T09:00:00+09:00",
			"ValidityStartDate": "2016-06-20T09:00:00+09:00",
			"ValidityEndDate": "9999-12-31T09:00:00+09:00",
			"ClassLastChangedDateTime": "",
			"KeyDate": "2022-01-27T09:00:00+09:00",
			"to_ClassCharacteristic": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_CLFN_CLASS_SRV/A_ClfnClassForKeyDate('1')/to_ClassCharacteristic",
			"to_ClassDescription": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_CLFN_CLASS_SRV/A_ClfnClassForKeyDate('1')/to_ClassDescription"
		}
	],
	"time": "2022-01-27T17:19:24+09:00"
}
```

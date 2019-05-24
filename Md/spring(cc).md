＠リポジトリ実装側のコード。
public List<String> get(){
    StringBuilder.appendでsql文作成
    戻り値はList<String>型
    sql は、hibernateのsessionクラスのcreateSQLQuery()にsetParmeterでメソッドチェーンして最後にlist()メソッドでList化。
}

Gsonbuilder

hibernate

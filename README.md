# mysql-schema-sync

Ferramenta de sincronização automática de estrutura de tabela MySQL (atualmente suporta apenas sincronização de campos e índices, funções avançadas como particionamento ainda não são suportadas)

Ferramenta de sincronização automática do MySQL Schema  

Usado para sincronizar **alterações** do esquema do banco de dados `online` para o `ambiente de teste local`!
Apenas o esquema está sincronizado, os dados não estão sincronizados.

支持功能：  

1. Sincronizar **nova tabela**
2. Sincronizar alterações de **campos**: novo, modificado
3. Sincronizar alterações de **índice**: novo, modificado
4. Suporte **Visualização** (compare apenas alterações assíncronas)
5. Notificação por **e-mail** dos resultados da alteração
6. Suporta atualizações de blindagem **tabelas, campos, índices, chaves estrangeiras**
7. Suporta mais tabelas, campos, índices e chaves estrangeiras locais do que online
8. Com base neste projeto, o problema de que as operações subsequentes serão encerradas se uma tabela de partição for encontrada durante o processo de comparação é corrigido. Para tabelas de partição, outras alterações além das partições serão sincronizadas.
9. O suporte para que cada ddl execute apenas uma única modificação, visando ser compatível com problemas de ddl tidb A alteração de multi esquema não suportada, controlada pelo campo single_schema_change, está desativada por padrão.

## Instalar

```bash
go install github.com/hidu/mysql-schema-sync@master
```

## Configuração

Consulte o arquivo de configuração padrão config.json para configurar os endereços de origem e destino de sincronização.
Modificar destinatários de email Você pode receber notificações por email quando a operação falhar ou a estrutura da tabela for alterada.  

Por padrão, tabelas, campos, índices e chaves estrangeiras extras não serão excluídos. Se precisar excluir campos, índices e chaves estrangeiras, você pode usar o parâmetro `-drop`.

Exemplo de configuração (config.json):

```
{
      //source：fonte de sincronização
      "source":"test:test@(127.0.0.1:3306)/test_0",
      //dest：Banco de dados a ser sincronizado
      "dest":"test:test@(127.0.0.1:3306)/test_1",
      //alter_ignore： Campos e índices ignorados durante a sincronização
      "alter_ignore":{
        "tb1*":{
            "column":["aaa","a*"],
            "index":["aa"],
            "foreign":[]
        }
      },
      //  tables: table to check schema,default is all.eg :["order_*","goods"]
      "tables":[],
      //  tables_ignore: table to ignore check schema,default is Null :["order_*","goods"]
      "tables_ignore": [],
      //Quando há uma alteração ou falha, o destinatário do e-mail
      "email":{
          "send_mail":false,
         "smtp_host":"smtp.163.com:25",
         "from":"xxx@163.com",
         "password":"xxx",
         "to":"xxx@163.com"
      }
}
```

### Descrição do item de configuração JSON

source: 数据库同步源  
dest:   待同步的数据库  
tables： 数组，配置需要同步的表，为空则是不限制，eg: ["goods","order_*"]  
alter_ignore： 忽略修改的配置，表名为tableName，可以配置 column 和 index，支持通配符 *  
email ： 同步完成后发送邮件通知信息  
single_schema_change：是否每个ddl只执行单个修改

### 运行

### Execute diretamente

```shell
mysql-schema-sync -conf mydb_conf.json -sync
```

### Visualizar e gerar alteração sql

```shell
mysql-schema-sync -conf mydb_conf.json 2>/dev/null >db_alter.sql
```

### Usar agendamento de shell

```shell
bash check.sh
```

每个json文件配置一个目的数据库，check.sh脚本会依次运行每份配置。
log存储在当前的log目录中。

### Operação programada automaticamente

添加crontab 任务

```shell
30 ****  cd /your/path/xxx/ && bash check.sh >/dev/null 2>&1
```

### Descrição do parâmetro

```shell
mysql-schema-sync [-conf] [-dest] [-source] [-sync] [-drop]
```

Ilustrar：

```shell
mysql-schema-sync -help  
  -conf string
        配置文件名称
  -dest string
        待同步的数据库 eg: test@(10.10.0.1:3306)/test_1
        该项不为空时，忽略读入 -conf参数项
  -drop
        是否对本地多出的字段和索引进行删除 默认否
  -http
        启用web站点显示运行结果报告的地址，如 :8080,默认否
  -source string
        mysql 同步源,eg test@(127.0.0.1:3306)/test_0
  -sync
        是否将修改同步到数据库中去，默认否
  -tables string
        待检查同步的数据库表，为空则是全部
        eg : product_base,order_*
  -single_schema_change
        生成 SQL DDL 语言每条命令是否只会进行单个修改操作，默认否
```

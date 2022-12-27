## redis 相关操作

#### 连接

redis-cli 连接远程 redis 服务：
`redis-cli -h 192.168.0.1 -p 6379`


#### 操作

key 模糊查询：
`keys aaa?`


get：
`get aaa`


del:
`del aaa`


#### 冷知识

如果 key 当中含有空格，则使用 redis-cli 操作的时候要使用：
`get aaa\ bbb`

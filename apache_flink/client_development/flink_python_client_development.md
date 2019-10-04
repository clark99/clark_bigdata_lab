# 1 flink开发环境安装
下载git clone https://github.com/apache/flink
## 1.1 cmd命令行执行：
set  MAVEN_OPTS="-Xmx4G"
mvn clean install package -Dmaven.test.skip=true
## 1.2 powershell命令行执行
set  MAVEN_OPTS="-Xmx4G"
mvn clean install package '-Dmaven.test.skip=true'
## 1.3 flink的目录结构
```shell
cd E:\app-installtools\flink\flink-dist\target\flink-1.9-SNAPSHOT-bin\flink-1.9-SNAPSHOT\bin
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        2019/4/24     11:50          29647 config.sh
-a----        2019/4/24     11:50           2279 flink
-a----        2019/4/24     11:50           2847 flink-console.sh
-a----        2019/4/24     11:50           6545 flink-daemon.sh
-a----        2019/4/24     11:50           1271 flink.bat
-a----        2019/4/24     11:50           1603 historyserver.sh
-a----        2019/4/24     11:50           2967 jobmanager.sh
-a----        2019/4/24     11:50           1849 mesos-appmaster-job.sh
-a----        2019/4/24     11:50           1883 mesos-appmaster.sh
-a----        2019/4/24     11:50           1935 mesos-taskmanager.sh
-a----        2019/4/24     11:50           1207 pyflink-stream.sh
-a----        2019/4/24     11:50           1166 pyflink.bat
-a----        2019/4/24     11:50           1132 pyflink.sh
-a----        2019/4/24     11:50           3517 sql-client.sh
-a----        2019/4/24     11:50           2597 standalone-job.sh
-a----        2019/4/24     11:50           3364 start-cluster.bat
-a----        2019/4/24     11:50           1889 start-cluster.sh
-a----        2019/4/24     11:50           3538 start-scala-shell.sh
-a----        2019/4/24     11:50           1900 start-zookeeper-quorum.sh
-a----        2019/4/24     11:50           1663 stop-cluster.sh
-a----        2019/4/24     11:50           1891 stop-zookeeper-quorum.sh
-a----        2019/4/24     11:50           3941 taskmanager.sh
-a----        2019/4/24     11:50           1714 yarn-session.sh
-a----        2019/4/24     11:50           2346 zookeeper.sh
```
## 1.4 启动flink
```shell
E:\app-installtools\flink\flink-dist\target\flink-1.9-SNAPSHOT-bin\flink-1.9-SNAPSHOT\bin\start-cluster.bat
Web interface by default on http://localhost:8081/.
```

# 2 flink计算word count案例
## 2.1 word count案例1
```python
from flink.plan.Environment import get_environment
from flink.functions.GroupReduceFunction import GroupReduceFunction

class Adder(GroupReduceFunction):
  def reduce(self, iterator, collector):
    count, word = iterator.next()
    count += sum([x[0] for x in iterator])
    collector.collect((count, word))

env = get_environment()
data = env.from_elements("Who's there?",
 "I think I hear them. Stand, ho! Who's there?")

data \
  .flat_map(lambda x, c: [(1, word) for word in x.lower().split()]) \
  .group_by(1) \
  .reduce_group(Adder(), combinable=True) \
  .map(lambda y: 'Count: %s Word: %s' % (y[0], y[1])) \
  .output()

# Out[6]: <flink.plan.DataSet.DataSink at 0x1fa0f6e42b0>
env.execute(local=True)
```
### 2.2 代码输出结果为
```
Count: 2 Word: i
Count: 1 Word: ho!
Count: 1 Word: hear
Count: 1 Word: them.
Count: 1 Word: think
Count: 2 Word: who's
Count: 1 Word: stand,
Count: 2 Word: there?
```

## 2.3 代码讲解
### 2.3.1 Map
输入一个元素，输出一个元素 
```python
data.map(lambda x: x * 2)
```
### 2.3.2 FlatMap 
输入一个元素，输出0个,1个,或多个元素
```python
data.flat_map(
  lambda x,c: [(1,word) for word in line.lower().split() for line 
in x])
```
### 2.3.3 MapPartition
- 通过一次函数调用实现并行的分割操作。
- 该函数将分割变换作为一个”迭代器”，并且能够产生任意数量的输出值。
- 每次分割变换的元素数量取决于变换的并行性和之前的操作结果。
```python
data.map_partition(lambda x,c: [value * 2 for value in x])
```
### 2.3.4 Filter
对每一个元素，计算一个布尔表达式的值，保留函数计算结果为true的元素。
```python
data.filter(lambda x: x > 1000)
```
### 2.3.5 Reduce
- 通过不断的将两个元素组合为一个，来将一组元素结合为一个单一的元素。
- 这种缩减变换可以应用于整个数据集，也可以应用于已分组的数据集。
```python
data.reduce(lambda x,y : x + y)
```
### 2.3.6 ReduceGroup
- 将一组元素缩减为1个或多个元素。
- 缩减分组变换可以被应用于一个完整的数据集，或者一个分组数据集。
```python
class Adder(GroupReduceFunction):
  def reduce(self, iterator, collector):
    count, word = iterator.next()
    count += sum([x[0] for x in iterator)      
    collector.collect((count, word))

data.reduce_group(Adder())
```

## 2.4 代码优化,结果写入到文件中
```python
from flink.plan.Environment import get_environment
from flink.plan.Constants import INT, STRING, WriteMode
from flink.functions.GroupReduceFunction import GroupReduceFunction

class Adder(GroupReduceFunction):
  def reduce(self, iterator, collector):
    count, word = iterator.next()
    count += sum([x[0] for x in iterator])
    collector.collect((count, word))

env = get_environment()
data = env.from_elements("Who's there?",
 "I think I hear them. Stand, ho! Who's there?")
output_file = 'file:///../examples/out.txt'

data \
 .flat_map(lambda x, c: [(1, word) for word in x.lower().split()]) \
 .group_by(1) \
 .reduce_group(Adder(), combinable=True) \
 .map(lambda y: 'Count: %s Word: %s' % (y[0], y[1])) \
 .write_text('out.txt', write_mode=WriteMode.OVERWRITE)

# Out[6]: <flink.plan.DataSet.DataSink at 0x1fa0f6e42b0>

env.execute(local=True)
```
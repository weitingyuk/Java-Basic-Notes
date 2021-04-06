## 简述一致性哈希算法的实现方式及原理

### 一致性哈希原理
- 1.结构：
    - 整个哈希值空间组织成一个虚拟的圆环，如假设某哈希函数H的值空间为0-2^32-1（即哈希值是一个32位无符号整形），整个哈希空间环按顺时针方向组织。0和232-1在零点中方向重合。
- 2.将服务器放置到哈希环：
    - 将各个服务器（t1, t2,t3... t10）使用Hash进行一个哈希，具体可以选择服务器的ip或主机名作为关键字进行哈希，这样每台机器就能确定其在哈希环上的位置。
- 3.将对象放置到哈希环：
    - 接下来使用如下算法定位数据访问到相应服务器：将数据key使用相同的函数Hash计算出哈希值，并确定此数据在环上的位置
- 4.为对象选择服务器：
    - 将对象和服务器都放置到同一个哈希环后，在哈希环上顺时针查找距离这个对象的 hash 值最近的机器，即是这个对象所属的机器。 
- 5.服务器增加的情况：
    - 假设由于业务需要，我们需要增加一台服务器 cs4，经过同样的 hash 运算，该服务器最终落于 t1 和 t2两台服务器之间
    - 这种情况，只有 t1和t2服务器之间的对象需要重新分配。
    - 好处：如果使用简单的取模方法，当新添加服务器时可能会导致大部分缓存失效，而使用一致性哈希算法后，这种情况得到了较大的改善，因为只有少部分对象需要重新分配。
- 6.服务器减少的情况：
    - 假设 t3 服务器出现故障导致服务下线，这时原本存储于 t3 服务器的对象 o4，需要被重新分配至 t2 服务器，其它对象仍存储在原有的机器上。
- 7.虚拟节点：
    - 问题： case 5 中新增的服务器cs4只分担了 t2 服务器的负载，其他的服务器并没有因为 cs4 服务器的加入而减少负载压力。
    - 解决方案：可以通过引入虚拟节点来解决负载不均衡的问题。即将每台物理服务器虚拟为一组虚拟服务器，将虚拟服务器放置到哈希环上，如果要确定对象的服务器，需先确定对象的虚拟服务器，再由虚拟服务器确定物理服务器。

### 一致性哈希作用
一致性哈希算法，在移除或者添加一个服务器时，能够尽可能小地改变已存在的服务请求与处理请求服务器之间的映射关系。


### 一致性哈希实现
使用TreeMap来保存虚拟节点到真实节点的map;
使用HashMap来保存真实节点到虚拟节点的map;

```
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.SortedMap;
import java.util.TreeMap;

public class ConsistenceHashWithTreeMap {

  private int virtualNums = 0;
  private Map<String, List<String>> real2VirtualMap = new HashMap<>();
  private SortedMap<Integer, String> sortedMap = new TreeMap<>();

  private List<String> realNodes = new ArrayList<>();

  public ConsistenceHashWithTreeMap(int virtualNums) {
    this.virtualNums = virtualNums;
  }

  public void addServer(String node) {
    realNodes.add(node);
    String vNode = null;
    List<String> virtualNodes = new ArrayList<>(this.virtualNums);
    this.real2VirtualMap.put(node, virtualNodes);
    for (int i=0; i<this.virtualNums; i++) {
      vNode = node + "vvv&&vv" + i;
      virtualNodes.add(vNode);
      int hashValue = getHash(vNode);
      this.sortedMap.put(hashValue, node);
    }
  }

  public String getServer(String key) {
    int hashValue = getHash(key);
    SortedMap<Integer, String> subMap = sortedMap.tailMap(hashValue);
    if (subMap.isEmpty()) {
      return sortedMap.get(sortedMap.firstKey());
    } else {
      return subMap.get(subMap.firstKey());
    }
  }

  private static int getHash(String str)
  {
    final int p = 16777619;
    int hash = (int)2166136261L;
    for (int i = 0; i < str.length(); i++)
      hash = (hash ^ str.charAt(i)) * p;
    hash += hash << 13;
    hash ^= hash >> 7;
    hash += hash << 3;
    hash ^= hash >> 17;
    hash += hash << 5;

    if (hash < 0)
      hash = Math.abs(hash);
    return hash;
  }

  public static void main(String[] args) {
    ConsistenceHashWithTreeMap ch = new ConsistenceHashWithTreeMap(10);
    ch.addServer("192.168.0.11");
    ch.addServer("192.168.0.12");
    ch.addServer("192.168.0.13");
    for (int i=0; i<10; i++) {
      System.out.println("The server of treeMap " + i + " is " + ch.getServer("treeMap " + i));
    }
  }
}
```

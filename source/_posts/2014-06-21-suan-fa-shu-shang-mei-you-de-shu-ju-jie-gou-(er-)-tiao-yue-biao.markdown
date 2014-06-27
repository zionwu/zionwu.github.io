---
layout: post
title: "算法书上没有的数据结构（二）-跳跃表"
date: 2014-06-21 05:00
comments: true
categories: 
---

##跳跃表
###什么是跳跃表
跳跃表是一种基于链表的数据结构，它支持对有序序列的快速查找。它在工程领域中有着广泛的应用，如[redis](en.wikipedia.org/wiki/Redis)和[leveldb](en.wikipedia.org/wiki/LevelDB)都是它作为底层数据结构。下图是一个跳跃表的例子：    
{% img center ./../images/skiplist/skiplist.png %}   

如图所示，该跳跃表有一个表头以及10个节点。节点之间并非只有一个链接，而是有多个链接。上层链接间的节点距离较远，而底层链接间的节点距离较近。最下面一层的链接距离为1，就是普通的链表。每次查找节点的时候，都是从最上层的节点开始查找，这样就**跳跃**了中间的节点了。下面一节将详细讲解跳跃表的实现。


###跳跃表的操作
跳跃表的实现并不复杂。与普通的链表节点不同，跳跃表的节点中包含了多个对后续节点的链接，我们只要弄清楚如何处理这些链接，就能够知道如何构建和操作跳跃表了。对一个跳跃表插入新元素的算法描述如下：
> 往跳跃表sl插入新的元素v。将节点n设置为表头节点，将层次l设置为最高层m，初始化一个大小为m的数组hist，做以下操作：     

> 1. 如果这一层的链接为空，那么l指向同一节点的下一层。   
> 2. 如果这一层的链接p非空，并且p指向的节点pn的值小于v，那么将节点n设置为pn。    
> 3. 如果这一层的链接p非空，并且p指向的节点pn的值大于v，那么将hist[l]设置为当前节点n, l指向同一节点的下一层。    
> 4. 如果这一层的链接p非空，并且p指向的节点pn的值等于v，那么该值已经存在于跳跃表中了。

> 当l为1(最低层)，并且当前节点n的链接指向节点的值大于v时，我们结束查找。将v插入到节点n后面，对于数组hist的节点hist[i]，调整该节点在第i层的链接。

下面的图片是一个具体的例子：    
{% img center ./../images/skiplist/skiplist-add.gif %}   

如题所示，往表里插入80，从表头的最高层开始。这一层的链接p为空， 因此层次l指向下一层，即第三层。第三层的链接p非空，而链接指向的节点值为50，比80小，因此当前节点前进至p所指节点。该节点在第三层的链接为空，所以层次l指向下一层，即第二层。第二层链接p非空，指向节点的值为70, 比80小，当前节点前进至下一节点。该节点第二层的链接为空，则层次l指向下一层，即第一层。在第一层，下一个节点的值为90, 比80大，因此结束查找，在70与90的节点中插入值为80的新节点。对于新节点，其有效的层次可以随机指定，即rand()%4，假设随机值是2, 那么我们需要调整第一层和第二层最后经过节点的链接。这就是为什么我们需要使用数组hist保存每一层最后经过的节点。

###跳跃表的实现
我们以redis的skiplist的源码为例子，讲解skiplist的实现。 redis中skiplist的数据结构的定义如下：   

    typedef struct zskiplist {
        //保存了表头和表尾
        struct zskiplistNode *header, *tail;    
        //元素的数量
        unsigned long length;    
        //最高层的值
        int level;     
    } zskiplist;     
    
    typedef struct zskiplistNode {
        //redis对象，保存具体的值
        robj *obj;
        //用来表示节点的大小
        double score;
        //指向前一个节点
        struct zskiplistNode *backward;
        //多层的链接
        struct zskiplistLevel {
            //下一个节点
            struct zskiplistNode *forward;
            //本节点与下一个节点的距离
            unsigned int span;
        } level[];
    } zskiplistNode;

zskiplist结构保存了指向表头表尾。注意表头是一个特殊节点，并不保存数据，而表尾则保存着实际的值。zskiplistNode结构表示一个节点，redis使用score字段来表示节点的大小，如果该字段相等，则在比较redis对象robj的大小。节点中有一个数组level, 该数组表示了多层的链接，每层链接除了指向下一个节点的指针，还有与下一节点的距离。

接下来我们看看redis中如何往skiplist插入一个新的节点：  

    //往skiplist中插入一个新节点
    zskiplistNode *zslInsert(zskiplist *zsl, double score, robj *obj) {
        zskiplistNode *update[ZSKIPLIST_MAXLEVEL], *x;
        unsigned int rank[ZSKIPLIST_MAXLEVEL];
        int i, level;

        redisAssert(!isnan(score));
        x = zsl->header;
    
        //先从高层找起，即level的zsl->level-1到0
        for (i = zsl->level-1; i >= 0; i--) {
            /* store rank that is crossed to reach the insert position */
            //将上一层跳过的元素数量加到这一层的rank中
            rank[i] = i == (zsl->level-1) ? 0 : rank[i+1];
            //如果这一层forward指针不为空，而且指向的元素比插入元素小，继续在该层上前进
            while (x->level[i].forward &&
                (x->level[i].forward->score < score ||
                    (x->level[i].forward->score == score &&
                    compareStringObjects(x->level[i].forward->obj,obj) < 0))) {
                //将跳过元素数量加到rank中
                rank[i] += x->level[i].span;
                //指向下一个节点
                x = x->level[i].forward;
            }
            //记录这一层上最后一个小于插入值的节点
            update[i] = x;
        }
        /* we assume the key is not already inside, since we allow duplicated
        * scores, and the re-insertion of score and redis object should never
        *  happen since the caller of zslInsert() should test in the hash table
        * if the element is already inside or not. */
        level = zslRandomLevel();
        //如果随机得到的level比当前skip的level要大
        if (level > zsl->level) {
            for (i = zsl->level; i < level; i++) {
                rank[i] = 0;
                //更新从zsl->level到level的update值
                update[i] = zsl->header;
                //更新update指向节点的level[i].span的值
                update[i]->level[i].span = zsl->length;
            }
            zsl->level = level;
        }
        x = zslCreateNode(level,score,obj);
        for (i = 0; i < level; i++) {
            //将新节点第i层的forward指向update[i]节点第i层的forward
            x->level[i].forward = update[i]->level[i].forward;
            //将update[i]节点第i层的forward指向新的节点
            update[i]->level[i].forward = x;

            /* update span covered by update[i] as x is inserted here */
            //计算新的节点的span值。rank[0] - rank[i]为新元素到update[i]的距离。
            //update[i]->level[i].span为该节点到原下一节点的距离。
            //相减则为新节点到下一节点的距离
            x->level[i].span = update[i]->level[i].span - (rank[0] - rank[i]);
            //更新update[i]->level[i].span 
            update[i]->level[i].span = (rank[0] - rank[i]) + 1;
        }

        /* increment span for untouched levels */
        //对于level到zsl->level的节点，它们的forward指针并没有改变，只需要对span值加1
        for (i = level; i < zsl->level; i++) {
            update[i]->level[i].span++;
        }

        //更新backward指针
        x->backward = (update[0] == zsl->header) ? NULL : update[0];
        if (x->level[0].forward)
            x->level[0].forward->backward = x;
        else
            zsl->tail = x;
        zsl->length++;
        return x;
    }    
    
该函数按照上一节算法描述，从表头最高层开始，找到新节点要插入的位置，同时记录每一层次上最后经过的节点。然后初始化一个新的节点，随机得到该节点的有效链接层数，然后将节点插入并更新各层的链接。   

对redis源代码感兴趣的读者，可以参考我对redis 2.8.9代码添加注释的[repos](https://github.com/zionwu/redis-2.8.9-annotation)。

###总结
跳跃表并不是一个复制的数据结构，但是它的各种操作都十分高效。其插入，查询和删除操作的平均时间复杂度都是O(logN)。因此如果在应用中需要一个有序集合，不妨考虑使用跳跃表。

####参考资料
1.  http://en.wikipedia.org/wiki/Skip_list
2.  Pugh, W. (1990). "Skip lists: A probabilistic alternative to balanced trees". Communications of the ACM 33 (6): 668. doi:10.1145/78973.78977
3.  [redis源代码](https://github.com/antirez/redis)





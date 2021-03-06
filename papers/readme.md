一般的适应拓扑变化的方案是多智能体强化学习，类似于Q-Routing方案，这类方案本身就没有拓扑的完整信息，每一个智能体通过局部信息来判断当前最优决策。这类方案的缺点在于局部最优不一定是全局最优，部署困难开销较大等。

论文Deep Reinforcement Learning meets Graph Neural Networks: An optical network routing use case中给出了DRL+GNN的方案。该方案使用DQN算法，Q值网络由GNN建模。
     state:当前网络状态（原文中以OTN网络为例），流量需求<src, dst, bandwidth>
     action：当前流量的路由策略
     reward：并没有详细介绍
从该方案中得出结论，使用GNN直接输入拓扑图的策略应用与强化学习算法是可行的

另外，GNN网络的输入是一个图，表示为G=(V，E，W)，V为节点集，E为边集，W为链路权重，可以通过链路带宽、时延等QoS参数设计。

传统路由算法ECMP是通过五元组信息进行哈希，根据哈希结果选择相应的转发端口进行数据转发。在对称拓扑上可以被认为是流粒度下的“最优解”。而为了适应非对称和可变的拓扑，由于网络设备的异构性或设备故障的发生,会导致网络产生不对称的情况;同时,数据中心中由于不同功能的服务器在机架上的部署,使得网络中的流量不是均匀分布的。
WCMP 对 ECMP 机制进行改进,利用权重对流哈希产生的交换机端口进行权重选择.网络中利用中央控制器进行网络状态感知,当发现网络出现不平衡性时,中央控制器计算出最优的权重分配并发送给相应交换机,交换机本地修改哈希权重执行数据包转发.WCMP 可以通过修改权重的方式来应对各种网络不平衡的情况。

Hedera利用一个中央控制器从边界交换机中收集网络的流量信息,并分析网络中存在的大流以实现大流的调度,从而实现网络的负载均衡。从边界交换机获取的流量信息迭代地计算各边界交换机对间所有流的稳态带宽需求,并将超过链路带宽一定阈值的流视为大流。调度器只能实现 5s 周期的调度,使其无法处理流量变化迅速的网络场景。（用流量信息估算再计算需求适应网络变化这种方法太慢了），如MicroTE等方案也采用这类方法（收集流信息并进行集中调度）。

MRL—MPRP方案（原文以传感器网络为例）每一个节点的状态可以表示为链路质量和邻居队列长度等。动作选择是下一跳节点。所以，我们可以选择把链路信息等作为强化学习智能体的输入，随时感知链路信息和拓扑信息。另外，在GNN方案中可以作为计算链路权重集合W的参数。

QoS模型指标：1.链路利用率ue。链路利用率表示分配在链路 e 上的流量大小与链路带宽的比值,反映了链路的负载程度。
2. 网络使用率 U 。网络使用率定义为各链路利用率的最大值。
3. 负载均衡程度 σ 。用所有链路中链路利用率的最大值和最小值之差表示链路的负载均衡程度。
状态(state)：在此模型中,状态是指某一次网络测量时网络中的流请求信息和所有链路的时延和利用率信息,用向量 s t 表示,则 s t = [ D t , l t , u t ] ,其中 D t 表示 t 时刻的流请求矩阵, l t 表示 t 时刻各节点对之间的传输时延, u t 示 t 时刻网络中各链路的链路利用率。（这就是拓扑信息加入输入的方案，我认为也可以用来计算GNN输入中的W）
动作(action)。动作是指强化学习智能体根据QoS 策略函数和网络状态信息 s t 生成的上述优化问题的解,也就是各节点对之间可用转发路径的分流比重。
奖赏(reward)。奖赏是对上一次动作所获收益的 QoS 评价。本模型中,软件定义网络 QoS 优化模型的优化目标是最小化网络使用率 U
这个方案的对比方案：DDPG : DDPG 方案中不采用 LSTM 层,其余神经网络参数设置与本文所提 R-DRL 算法一致,用来验证 LSTM 层对 DDPG 算法改进效果。
OSPF : OSPF 即开路最短路径优先,依据该规则,网络会把数据流转发在长度最短的路径上,由于没有考虑链路的传输能力,个别链路容易陷入拥塞。
MCFCSP :多物网络流流约束最短路径方案将链路的传输能力作为约束条件,在保证网络不出现拥塞的条件下传输数据流。
KSP : k 路最短路径方案会在两节点对间选择前 k 条最短的路径作为路由路径对数据流完成转发操作,本次实验中取 k=4 。

我提出的方案就是从这里来的，action不再是权重，变成了阈值分割流量，对每个flowcell进行决策的流量分割比。

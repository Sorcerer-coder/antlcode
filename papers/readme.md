论文Deep Reinforcement Learning meets Graph Neural Networks: An optical network routing use case中给出了DRL+GNN的方案
该方案使用DQN算法，Q值网络由GNN建模。
     state:当前网络状态（原文中以OTN网络为例），流量需求<src, dst, bandwidth>
     action：当前流量的路由策略
     reward：并没有详细介绍
从该方案中得出结论，使用GNN直接输入拓扑图的策略应用与强化学习算法是可行的

另外，GNN网络的输入是一个图，表示为G=(V，E，W)，V为节点集，E为边集，W为链路权重，可以通过链路带宽、时延等QoS参数设计。


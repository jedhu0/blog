

# **链上价格发现的演进：深入解析联合曲线与Meteora的动态发行协议**

---

## **第一部分：算法流动性的基石**

### **第1节：联合曲线作为DeFi原语的简介**

#### **1.1. 超越订单簿：自动做市商的起源**

在金融市场的历史长河中，订单簿模型一直是价格发现的核心机制。它通过匹配买卖双方的出价和要价来促成交易。然而，将这一传统模型直接移植到去中心化的区块链环境中面临着巨大挑战。链上交易的延迟、交易成本（Gas费）以及流动性分散等问题，使得维持一个高效、深度的链上订单簿变得不切实际。传统的价格发现过程被描述为一个“不可预测且不稳定的过程”，常常受到中心化做市商的影响 1。

为了应对这些挑战，去中心化金融（DeFi）领域催生了一项革命性的创新：自动做市商（Automated Market Maker, AMM）。AMM放弃了传统的买卖方匹配模式，转而采用算法驱动的流动性池。用户不再是与另一个交易者进行点对点交易，而是与一个由智能合约管理的资金池进行交互。这一范式转变为在无需许可、无需信任的区块链上创建可持续的流动性市场奠定了基础，而联合曲线（Bonding Curve）正是实现这一范式转变的核心原语之一。

#### **1.2. 定义联合曲线：价格与供应量的算法链接**

联合曲线是一个数学函数，它通过智能合约实现，为一种代币的价格与其流通供应量之间建立了一种确定性的关系 2。与由买卖双方的博弈决定价格的订单簿不同，联合曲线中的代币价格完全由预设的算法决定。当用户购买代币时，价格沿曲线向上移动；当用户出售代币时，价格则沿曲线向下滑动。

在这个模型中，智能合约本身扮演了交易对手方的角色，它始终准备好根据曲线定义的当前价格来铸造（出售）或销毁（回购）代币 7。这种机制消除了对中心化中介或传统订单簿的需求，创建了一个自主、透明且可预测的代币经济系统。

#### **1.3. 核心机制：铸造、销毁与储备池范式**

联合曲线的运作流程可通过以下三个核心概念来理解：

* **铸造 (Minting/购买):** 当用户希望购买代币时，他们向联合曲线的智能合约发送一种指定的储备货币（例如SOL、ETH或USDC）。智能合约收到储备货币后，会根据曲线当前的数学公式计算出应铸造的新代币数量，并将其发送给用户。此操作增加了代币的总供应量，并导致下一个买家需要支付更高的价格 1。用户支付的储备货币则被智能合约锁定，作为抵押品存入储备池 1。  
* **销毁 (Burning/出售):** 当代币持有者希望出售其代币时，他们将代币发送回智能合约。合约接收到代币后会将其销毁，从而减少流通供应量。随后，合约会根据曲线当前的价格，从储备池中提取相应数量的储备货币支付给用户。此操作导致下一个卖家的出售价格降低 4。  
* **储备池 (Reserve Pool):** 这个由储备货币构成的资金池是联合曲线的价值支撑。它为所有已发行的代币提供了抵押，并保证了市场的持续流动性，因为理论上用户可以随时将其代币卖回给合约以换取储备资产 4。从数学角度看，储备池的规模等于联合曲线下方的面积，即价格函数从零到当前供应量的积分 1。

#### **1.4. 作为主要自动做市商（PAMM）的联合曲线**

在DeFi生态中，需要区分主要自动做市商（Primary Automated Market Maker, PAMM）和次级自动做市商（Secondary Automated Market Maker, SAMM）。像Uniswap这样的平台属于SAMM，它们为已经存在的代币对提供交易场所。而联合曲线则扮演着PAMM的角色：它从零开始创造代币，并为其建立一个全新的市场 4。

这种机制从根本上解决了新项目面临的“冷启动”流动性难题。传统市场依赖专业的做市商提供初始流动性，这是一项资本密集型业务。标准的AMM（如Uniswap）虽然通过允许任何人成为流动性提供者（LP）来降低门槛，但仍要求LP以特定比例（通常是50:50的价值比）存入两种资产 10。这意味着一个新项目方需要持有大量稳定币或主流资产，才能为其新代币创建一个有深度的流动性池。

联合曲线则巧妙地绕过了这一障碍。项目方只需部署一个智能合约，而无需提供任何初始资本。第一个购买者用储备货币（如SOL）买入第一个代币，从而提供了第一笔储备金 1。流动性由市场需求单向启动，智能合约通过算法自动定价并按需铸造代币，充当了交易的另一方。因此，联合曲线不仅是订单簿的替代品，更是一种资本效率极高的流动性引导机制，尤其适用于草根项目和需要持续融资的模式 3。

### **第2节：[联合曲线](https://g.co/gemini/share/6e2ddd73ad29)的数学与经济架构**

#### **2.1. 曲线剖析：常见数学模型分析**

联合曲线的设计空间极为广阔，不同的数学公式会产生截然不同的经济激励和行为模式。以下是几种最常见的模型：

* **线性曲线 (Linear Curve):** 其价格函数为 Price=m⋅Supply+b。在这个模型中，m 是斜率，b 是初始价格。代币价格随着供应量的增加而匀速增长。这种曲线的优点是价格走势可预测、增长平稳，但对早期参与者的激励相对较弱。它更适用于追求稳定发展的社区代币或项目 8。  
* **指数曲线 (Exponential Curve):** 价格函数通常为 Price=a⋅e(k⋅S) 或更简单的多项式形式 Price=Supplyn。在这种模型下，价格起初增长缓慢，但随着供应量的增加会急剧加速。这种设计极大地奖励了早期采用者，并能有效吸引投机性需求，非常适合由热度驱动的代币发行。然而，其缺点是可能导致剧烈的价格波动，并对后期进入者形成高门槛 13。  
* **对数曲线 (Logarithmic Curve):** 价格函数为 Price=k⋅ln(S+c)。价格在早期供应量较低时迅速上涨，但随着供应量的增加，价格上涨速度逐渐放缓并趋于平稳。这种模型适合那些希望在早期快速引导流动性，然后在发展后期稳定价格以促进广泛采用的项目 3。  
* **其他模型:** 除了上述三种，还存在更复杂的曲线设计，如多项式曲线（P=a⋅Sn）、S型曲线（Sigmoid Curve，适用于具有明显生命周期的项目，模拟慢-快-平稳的增长阶段）和阶梯函数曲线（Step Function，价格在达到特定里程碑时跳跃式增长），这些模型为代币经济设计提供了极大的灵活性 13。

#### **2.2. 多代币兑换的定价：积分学的应用**

一个关键的技术细节是，当用户一次性购买大量代币时，其总成本并非简单地用当前单价乘以购买数量。由于联合曲线的特性，每铸造一个（甚至是无穷小的）代币，价格都会发生变化。因此，精确的总成本是通过对价格函数进行积分运算得出的，即计算从起始供应量到结束供应量这段区间内，曲线下方的面积。这个概念对于理解智能合约的实现方式和交易中的滑点至关重要 1。

#### **2.3. Bancor公式与储备金率（RR）**

Bancor协议的公式为联合曲线提供了一个更具普适性的分析框架。其核心是储备金率（Reserve Ratio, RR），也称为连接器权重（Connector Weight）。

* **公式:** Reserve Ratio=Token Supply×Token PriceReserve Token Balance​ 19。  
* **含义:** 储备金率是一个介于0%到100%之间的常数，它决定了代币价格对供应量变化的敏感度，即价格的波动性。较低的储备金率（如10%）会产生一条陡峭的、类似指数的曲线，价格随供应量变化而剧烈波动。较高的储备金率（如50%或更高）则会产生一条平缓的、类似线性的曲线，价格敏感度较低 7。储备金率是代币经济设计者手中一个强有力的调节杠杆。

#### **表1：联合曲线模型对比分析**

为了直观地比较不同曲线模型，下表总结了它们的关键特征。

| 曲线类型 | 数学公式 | 价格行为 | 激励结构 | 主要用例 | 主要风险 |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **线性曲线** | P=mS+b | 稳定、可预测的线性增长 | 对所有参与者相对公平，早期激励较弱 | 社区代币、稳定增长项目 | 可能无法吸引投机者，增长潜力有限 |
| **指数曲线** | P=a⋅e(kS) 或 P=aSn | 初始缓慢，随后急剧加速 | 极大地奖励早期采用者 | 炒作驱动的发行、Meme币、NFT发行 | 波动性极高，对后期参与者不友好，可能导致泡沫 |
| **对数曲线** | P=k⋅ln(S+c) | 初始快速增长，随后趋于平缓 | 奖励早期参与者，但后期价格稳定 | 引导初始流动性，然后稳定价格以促进广泛采用 | 早期暴涨后可能失去增长动力，使新买家失去兴趣 |
| **S型曲线** | 复杂的S形函数 | 慢-快-慢的生命周期增长模式 | 平衡早期、成长期和成熟期参与者的利益 | 具有明确发展阶段的长期DAO或项目 | 数学和经济模型复杂，难以理解和信任 |
| **阶梯曲线** | 分段常数函数 | 在特定供应量里程碑处价格跳涨 | 奖励在关键发展节点前参与的用户 | 游戏化发行、与项目进展挂钩的NFT投放 | 价格跳跃可能引发抢先交易，波动不可预测 |

资料来源：综合整理自 3

深入思考，联合曲线的本质远不止一个定价工具，它实际上是一个项目被代码化的“货币政策”。传统世界的中央银行通过利率等工具调控货币供给，影响经济活动。在DeFi世界中，一个项目的联合曲线数学公式 2 及其关键参数（如储备金率 19）被硬编码到不可篡改的智能合约中 4。这个公式规定了代币的“发行政策”（如何铸造新币）和对需求的“利率响应”（价格如何变化）。一条指数曲线相当于一种“紧缩”的货币政策，资本成本（即代币价格）会迅速攀升；而一条线性曲线则是一种“宽松”、可预测的政策。正如Swarm项目的Gregor所指出的，这种设计在提供控制和可预测性的同时，牺牲了灵活性 21。项目从第一天起就承诺执行一项固定的货币政策。因此，分析一个项目的联合曲线，就等同于分析其内在的经济哲学——它揭示了项目方对增长的预期、目标受众（是投机者还是社区成员）以及长期的经济战略。

---

## **第二部分：应用、创新与内在风险**

### **第3节：联合曲线应用场景概览**

联合曲线作为一种灵活的DeFi原语，其应用已渗透到多个领域，不断推动着链上经济的创新。

* **3.1. 自动做市商:** 这是联合曲线最基础也是最广泛的应用。像Uniswap v2这样的协议所使用的恒定乘积公式（x⋅y=k）本质上就是一种特殊的联合曲线。它通过维持储备池中两种资产价值乘积的恒定，来动态调整价格并确保流动性 4。  
* **3.2. 去中心化治理与社区募资:** 去中心化自治组织（DAO）广泛采用联合曲线来发行其治理代币。这种模式能够将早期贡献者的利益与项目的长期成功紧密绑定，因为随着社区的发展和代币需求的增加，他们持有的代币价值也会随之上升。通过联合曲线销售代币所筹集的资金可以直接进入DAO的财库，形成一个自我维持的、由社区驱动的资金系统 1。  
* **3.3. 动态NFT定价与碎片化:**  
  * 在非同质化代币（NFT）领域，联合曲线打破了传统的静态定价和拍卖模式。像sudoswap这样的平台利用联合曲线为NFT创建即时、流动的市场。用户可以直接从一个池子中购买或出售NFT，其价格会根据预设的曲线参数（如线性或指数变化的delta）动态调整，极大地提高了NFT的交易效率和价格发现能力 22。  
  * 此外，联合曲线也被用于为碎片化的NFT（例如，将一个高价值NFT分割成多个ERC-20代币）提供定价和流动性。这解决了NFT衍生品流动性不足的核心痛点，通过保证这些“碎片”随时可以交易，从而促进了社区所有权和参与度 20。  
* **3.4. 持续与动态募资:** 联合曲线使项目能够摆脱一次性的首次代币发行（ICO）模式，转向一种持续的、按需的融资方式 3。更进一步的创新是“动态曲线”的出现，其形态可以根据外部条件变化而调整：  
  * **动态个体曲线:** 这种模型提出，曲线的形状可以取决于用户个人的持仓情况。例如，对于持有大量代币的用户，购买更多代币的成本会更高，而出售的价格也更高，反之亦然。这旨在激励代币的广泛分布 25。  
  * **里程碑驱动曲线:** 曲线的形态可以与项目的实际进展挂钩。例如，当项目通过预言机（Oracle）验证达成了某个重要里程碑后，曲线可以自动变得更加陡峭，以奖励那些在项目早期、风险更高时就已投入的信徒 8。  
  * **KPI驱动曲线:** 动态离散联合曲线（DDBC）模型提议，曲线的价格跳跃幅度和恒定价格区间的长度可以由协议的关键绩效指标（KPI）如收入或代币波动性来决定，从而将代币的经济模型与协议的实际表现直接关联 26。

联合曲线的演进路径清晰地反映了DeFi领域的宏观发展趋势：从静态、普适的机制，走向动态、自适应、具备状态感知的复杂系统。DeFi 1.0时代的协议（如Uniswap v2）以其简单、固定的数学模型为特征，早期的联合曲线概念亦是如此 2。而随着DeFi 2.0时代的到来，协议控制价值（PCV）、主动流动性管理（如Uniswap v3的集中流动性）等理念兴起。联合曲线也同样从固定的函数 6，演变为可根据KPI调整的动态模型 26，乃至像Meteora DBC那样高度可定制的分段式虚拟曲线 27。这表明，开发者们不再满足于“一刀切”的经济模型，而是致力于构建能够响应市场变化、协议表现和用户行为的、更精密的金融工具。这一演进趋势为理解Meteora DBC这类高级协议的诞生提供了必要的背景。

### **第4节：黑暗森林：MEV、漏洞利用与协议风险**

尽管联合曲线带来了诸多创新，但其透明和可预测的特性也使其暴露在区块链“黑暗森林”的各种威胁之下。

* **4.1. 经济与设计风险:** 最根本的风险源于曲线本身的设计不当。一条结构不合理的曲线可能导致价格不可持续、流动性枯竭或激励模型失效 3。此外，联合曲线模型的复杂性本身也可能成为普通用户理解和参与的障碍，从而影响项目的社区建设 21。  
* **4.2. MEV（最大可提取价值）：可预测性的悖论:** MEV是指交易排序者（如矿工或验证者）通过重新排序、插入或审查交易包而获得的额外利润 28。联合曲线的悖论在于，其透明和可预测的定价机制（优点）恰恰使其成为MEV攻击的理想目标（风险） 17。  
  * **抢先交易（Front-running）与三明治攻击（Sandwich Attacks）:** 这是最常见的漏洞利用方式。攻击者（通常是机器人）在公共内存池（mempool）中监测到一笔即将发生的大额购买交易。攻击者会立即以更高的Gas费提交自己的购买订单，确保其交易在受害者之前被执行（即“抢先交易”）。这笔交易推高了代币价格。随后，受害者的大额购买订单以这个被抬高的价格成交。紧接着，攻击者立即出售他们刚才购入的代币（即“尾随交易”或“Back-running”），从自己制造的价差中获利。受害者的交易被夹在攻击者的两笔交易之间，如同三明治一般，因此得名“三明治攻击” 31。  
* **4.3. 智能合约与协议层面的漏洞:**  
  * **闪电贷攻击 (Flash Loan Attacks):** 攻击者可以在一个原子交易内，从借贷协议中借出巨额资金（无需抵押），用这笔资金在联合曲线上进行一次大额购买，从而人为地操纵价格。然后，攻击者在另一个依赖此价格预言机的协议上执行有利可图的操作（如清算、套利），最后再将代币卖回联合曲线，偿还闪电贷并锁定利润 15。  
  * **预言机操纵 (Oracle Manipulation):** 如果联合曲线的参数（如价格）依赖于外部预言机提供的数据，那么攻击者就可能通过操纵预言机来影响曲线行为，从而获利 15。  
  * **算术精度与代码错误 (Arithmetic Bugs):** 在智能合约中实现复杂的数学公式时，整数溢出/下溢或舍入误差等编程错误可能导致严重的资金损失 36。采用经过严格审计和实战检验的数学库（如PRBMath）是关键的缓解措施 36。

如今，联合曲线设计的核心战场已经转移到了MEV的攻防上。早期的文献主要聚焦于经济模型和价格发现理论 2。然而，随着DeFi生态的成熟，后期和更具实践性的讨论则被MEV、抢先交易和三明治攻击等话题主导 17。这标志着行业关注点从理论的优雅性转向了现实世界的韧性。相应的，缓解策略也从单纯调整曲线形状，演变为关注交易动态本身。例如，引入提交-揭示（commit-reveal）方案、交易批处理 15、私人内存池 34，以及与本次研究密切相关的动态费用机制。动态费用，如基于时间的费用衰减或基于波动率的附加费，使得抢先交易的成本和收益变得不确定，从而有效抑制攻击行为 37。这表明，在当代DeFi环境中，一个“优秀”的联合曲线协议，不仅需要巧妙的数学公式，更需要一个能够在充满对抗性的公共内存池中生存和发展的稳健架构。这正是像Meteora DBC这样的协议的核心价值所在。

---

## **第三部分：案例研究 \- Meteora的动态联合曲线（DBC）**

### **第5节：协议深度剖析：Meteora DBC的架构与愿景**

#### **5.1. 面向Solana生态的无需许可发行协议**

Meteora动态联合曲线（Dynamic Bonding Curve, DBC）并非一个直接面向终端用户的应用程序，而是一个B2B（企业对企业）模式的基础设施层。它是一个无需许可的发行协议，旨在赋能*发行合作伙伴*（如Launchpad、DAO等平台），使其能够为*他们的*用户（即代币创建者）提供高度可定制化的代币发行服务 27。

#### **5.2. 端到端流程：从配置到AMM迁移**

Meteora DBC的完整生命周期包括以下几个关键步骤：

1. **配置 (Configuration):** 合作伙伴首先使用DBC的软件开发工具包（SDK）创建一个独特的config key。这个配置密钥封装了代币发行的所有参数，从曲线形状到最终的流动性迁移规则 39。  
2. **创建 (Creation):** 代币创建者在合作伙伴的平台上，利用这个config key来创建一种新代币和一个与之关联的DBC池 39。  
3. **启动与交易 (Launch & Trading):** 一旦DBC池创建成功，该新代币便可立即通过集成了DBC的平台进行交易，如聚合器Jupiter、交易终端Photon以及各类交易机器人 39。  
4. **毕业 (Graduation):** 当DBC池中积累的储备货币（如SOL）达到预设的migration\_quote\_threshold（迁移阈值）时，该池的交易功能将被暂停，准备进入下一阶段 39。  
5. **迁移 (Migration):** Meteora的自动化“迁移守护者”（migrator keeper）服务会“触发”（crank）该池，将其在联合曲线阶段募集到的流动性，自动迁移到一个永久性的Meteora动态AMM（DAMM v1或v2）池中 39。  
6. **LP管理 (LP Management):** 迁移后生成的流动性提供者（LP）凭证（通常是代币或NFT），会根据初始配置进行锁定或分配给合作伙伴和创建者。这使他们能够从新生成的AMM池中持续赚取交易手续费，分享项目成功的长期收益 27。

#### **5.3. 核心设计目标：定制化、可组合性与公平性**

Meteora DBC的设计旨在实现三大核心目标：为合作伙伴提供最大的灵活性来设计独特的发行体验；通过深度集成确保新代币从第一区块起就具备可交易性；以及提供先进工具来创造更公平的发行环境 39。

Meteora DBC的架构展现了一种深刻的行业洞察：它将复杂的代币发行过程“产品化”和“抽象化”，并将其转化为一种可配置的服务。传统上，使用联合曲线发行代币需要深厚的智能合约开发知识，涵盖数学、安全和代币经济学等多个领域。Meteora的架构 39 则将这些复杂性完全封装起来。合作伙伴无需编写任何Rust或Solidity代码来定义曲线，他们只需通过SDK与协议交互，在一个

config key中设置参数即可 27。这种模式将一项复杂的工程任务转变为一项配置任务，本质上是一种“联合曲线即服务”（Bonding-Curve-as-a-Service）。这极大地降低了Launchpad提供复杂、定制化代币发行服务的门槛，从而促进了Solana上发行生态的多样性和竞争力。这是一种典型的基础设施打法，通过赋能其他企业来推动整个生态系统的成熟。

### **第6节：Meteora DBC特性技术分析**

#### **6.1. 对抗MEV：反狙击技术详解**

Meteora DBC直接回应了第四节中详述的MEV威胁，内置了一套“反狙击套件”（Anti-Sniper Suite）：

* **费用调度器 (Fee Scheduler):** 这是一种基于时间的费用机制，在代币发行之初设置高额交易费，然后随时间推移（线性或指数性）逐渐降低。这种设计使得MEV机器人在最初几个区块或几秒钟内进行狙击的成本变得极其高昂，从而有效遏制抢先交易 37。  
* **动态费用 (Dynamic Fee):** 这是一种基于波动率的费用，在价格剧烈波动的时期自动提高。它在发行初期的炒作高峰期充当“高峰定价”，既能为LP捕获更多价值，又能抑制狙击者的攻击意愿 38。  
* **速率限制器 (Rate Limiter):** 这是一种基于交易金额的费用机制，交易额越大，费率越高。这旨在保护散户的小额交易免受狙击者大额购买所造成的巨大价格冲击 39。

#### **6.2. 前所未有的灵活性：可定制的流动性分布**

这是Meteora DBC的一项关键创新。与传统的平滑曲线不同，DBC允许合作伙伴通过定义多达20个离散价格区间的流动性分布，来构建一条分段式（piecewise）的虚拟曲线 27。其公式可以表示为

bonding\_curve=function(\[li​,pai​,pbi​\])，其中每个元组代表一个价格区间 \[pai​,pbi​\] 及其对应的流动性 li​ 27。这种设计使得项目方可以精细地控制价格曲线的形状，例如在初始阶段提供深厚的流动性以维持平坦的价格曲线，鼓励广泛分发，然后在价格较高阶段减少流动性，使曲线变得陡峭。这实际上是在联合曲线的框架内模拟了集中流动性（Concentrated Liquidity）的效果。

#### **6.3. 经济模型：费用、盈余与激励机制**

Meteora DBC设计了一套精巧的经济模型，以对齐各方利益：

* **交易费用:** 在DBC阶段产生的所有交易费用，会按照预设规则进行分配：一部分归Meteora协议，一部分作为推荐费分给交易入口（如Jupiter或交易机器人），剩余大部分归合作伙伴和代币创建者共享 39。  
* **迁移盈余 (Migration Surplus):** 当最后一笔交易使得池中储备金超过迁移阈值时，通常会产生一笔额外的储备代币盈余。这笔盈余将被分配：40%给创建者，40%给合作伙伴，20%给Meteora协议 39。这为合作伙伴和创建者创造了直接的经济激励，促使他们努力确保发行成功。  
* **LP激励:** 迁移到AMM池后，合作伙伴和创建者通过其锁定的LP头寸，可以持续从代币的长期交易中赚取手续费，从而将其利益与代币的长期健康发展绑定在一起 27。

#### **表2：Meteora DBC关键配置参数解析**

下表为希望集成Meteora DBC的合作伙伴提供了一份实用指南，将战略目标与具体参数设置联系起来。

| 参数 | 数据类型 | 描述 | 对Launchpad的战略意义 |
| :---- | :---- | :---- | :---- |
| pool\_fees | 结构体 | 定义基础费率、可选的动态费用和费用调度器 | 控制交易成本和MEV抵抗能力。启用动态费用可从发行初期的波动中捕获更多价值。 |
| migration\_quote\_threshold | u64 | 触发从DBC池到AMM池迁移的储备代币数量阈值 | 决定了初始募资阶段的规模。阈值越高，初始阶段越长，募集的初始流动性越多。 |
| curve | 数组 | 定义分段价格曲线，包含最多20个价格区间及其流动性 | 核心定制功能。可设计出任意形状的曲线，以实现特定的价格发现和代币分发策略。 |
| partner\_lp\_percentage | u64 | 迁移后，合作伙伴可领取的LP份额百分比 | 直接的经济激励。合作伙伴可以通过提供优质服务来协商更高的LP份额。 |
| creator\_locked\_lp\_percentage | u64 | 迁移后，为创建者永久锁定的LP份额百分比 | 增强社区信心。高比例的锁定LP表明创建者对项目的长期承诺，有助于防止“拉地毯”（Rug Pull）。 |
| locked\_vesting | 结构体 | 为创建者设定的LP份额的线性解锁计划 | 平衡创建者激励与市场稳定。防止创建者在发行后立即抛售大量代币，对市场造成冲击。 |
| token\_supply | u64 | 设定代币在迁移前后的固定总供应量 | 适用于非通胀模型。确保代币的稀缺性和可预测性。 |
| collect\_fee\_mode | u8 | 费用收集模式（0: 仅储备代币, 1: 两种代币） | 财务管理灵活性。例如，仅以SOL或USDC形式收取费用，可以简化财库管理。 |

资料来源：综合整理自 27

### **第7节：市场定位与竞争格局**

#### **7.1. Meteora DBC在Solana发行平台生态中的位置**

对Solana上代币发行市场的分析显示，Meteora DBC扮演着一个独特的基础设施角色。数据显示，与Pump.fun（36.9%）和新晋领导者LetsBonkFun（50.4%）等直接面向消费者的海量发行平台相比，Meteora DBC的直接市场份额仅为0.39% 45。这表面上看起来微不足道，但却揭示了其真正的市场定位：一个服务于其他发行平台的B2B基础设施提供商。

其竞争对手包括Raydium的LaunchLab，该平台同样提供定制化的联合曲线发行功能；以及Gavel，一个专注于通过荷兰式拍卖等机制实现MEV抵抗和公平发行的协议 43。

#### **7.2. 链上数据分析：采用率、交易量与费用产生**

尽管直接市场份额不高，但链上数据描绘了一幅截然不同的图景。根据DefiLlama的数据，Meteora DBC协议已处理了数亿美元的累计交易量，并产生了数千万美元的累计费用 46。更有数据显示，已有超过71,000种代币通过Meteora DBC创建 48。这些数据有力地证明，该协议被广泛采用，并正在创造巨大的经济价值。

#### **7.3. 集成的关键作用**

Meteora DBC的核心价值主张之一是“即时可交易性”。这得益于其与Solana生态内关键基础设施的深度集成，尤其是与交易聚合器龙头Jupiter以及各类主流交易机器人的无缝对接 39。当一个新代币通过DBC启动时，这些集成方可以立即识别并向其路由交易流。这种可组合性在节奏极快的Solana生态中是至关重要的竞争优势。

对市场数据的深入分析揭示了一个关键事实：Meteora DBC的成功不应以其直接市场份额来衡量，而应以其对整个生态系统的“赋能系数”来评估。初看之下，0.39%的市场份额 45 似乎表明其影响力有限。然而，一份报告中一个不起眼的细节彻底改变了这一看法：市场份额排名第一的发行平台，正是“完全基于Meteora的DBC构建的” 48。

这一发现将所有线索联系在一起。Meteora DBC的策略类似于科技行业的“Intel Inside”模式。它的品牌可能不为终端用户所熟知，但其技术却驱动着市场上最成功的应用。因此，其真正的成功指标并非其自有网站的用户数，而是其所赋能的合作伙伴平台的交易量、发行数量和整体成功。DefiLlama上显示的巨额交易量和费用数据 46，与极低的直接市场份额 45 之间的巨大反差，以及其技术被市场领导者采用的事实 48，共同印证了这一结论：Meteora DBC是Solana代币发行领域一个隐藏的、但至关重要的基础设施巨头。

---

## **第四部分：战略分析与未来展望**

### **第8节：比较框架：联合曲线 vs. 流动性引导池（LBP）**

为了更全面地理解联合曲线在代币发行领域的定位，有必要将其与另一种主流的公平发行机制——流动性引导池（Liquidity Bootstrapping Pool, LBP）进行比较。

#### **8.1. LBP机制：价格发现的荷兰式拍卖法**

LBP是一种利用动态权重调整来实现价格发现的资金池。其核心机制如下：

* **核心机制:** LBP通常由两种代币组成：项目代币和一种抵押代币（如USDC或SOL）。在LBP启动时，池子的权重被设定为极度偏向项目代币，例如99%的项目代币和1%的抵押代币。在预设的一段时间内（如72小时），池子的权重会自动、线性地向抵押代币方向调整，最终可能变为1:99 11。  
* **价格压力:** 这种权重的持续变化，为项目代币的价格施加了恒定的*下行压力*。这类似于荷兰式拍卖，价格从一个高点开始，不断下降，直到市场的购买压力足以抵消这种下行趋势，从而找到一个市场公认的均衡价格 52。

#### **8.2. 对比激励结构与公平哲学**

联合曲线和LBP体现了两种截然不同的发行哲学：

* **联合曲线:** 通常创造*上行*价格压力。价格随需求增加而上涨。其“公平”理念是“在同一时间点，所有人都能以相同的价格进行交易”。它奖励的是早期发现价值并抱有坚定信念的参与者 1。  
* **LBP:** 创造*下行*价格压力。其高昂的初始价格旨在有效阻止抢先交易的机器人和巨鲸（大户）在早期垄断供应。其“公平”理念是“抵抗市场操纵”，允许所有参与者从容地等待，直到价格达到他们认为合理的水平再入场 49。

#### **8.3. 选择合适的工具：战略用例**

* **何时使用联合曲线:** 适用于需要持续融资的项目、希望逐步增长的社区代币，或需要持续进行新NFT铸造的项目。  
* **何时使用LBP:** 适用于拥有固定供应量代币的一次性、公平启动活动，其核心目标是实现广泛的代币分配和有效的价格发现。

#### **表3：特性与策略比较：标准联合曲线 vs. LBP vs. Meteora DBC**

| 机制 | 价格压力 | 价格发现风格 | MEV抵抗性 | 资本效率 | 发行后状态 | 理想用例 |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| **标准联合曲线** | 上行 | 需求驱动型，价格随购买增加而上涨 | 较低（易受抢先交易攻击） | 极高（无需初始资本） | 在曲线上持续交易 | 持续融资，社区代币，动态NFT |
| **流动性引导池 (LBP)** | 下行 | 荷兰拍类型，价格随时间下降 | 较高（高初始价格抑制机器人） | 较高（项目方只需提供少量抵押资产） | 需手动创建AMM池 | 一次性公平发行，广泛分发 |
| **Meteora DBC** | 上行（可定制） | 需求驱动，但曲线形状高度可控 | 中到高（内置反狙击工具） | 极高（无需初始资本） | 自动迁移至永久AMM池 | 为Launchpad提供可定制、安全的代币发行服务 |

资料来源：综合整理自 39

这张对比表揭示了Meteora DBC在市场中的独特生态位。它在某种程度上是一个混合模型。它保留了传统联合曲线的上行价格压力和持续交易的特性，但通过内置的反狙击工具，吸收了LBP抵抗MEV的设计思想。最关键的是，它解决了LBP的一个核心痛点——发行后的流动性衔接问题。LBP结束后，项目方需要手动将其募集的资金与剩余代币配对，在DEX上创建流动性池，这个过程既有风险也容易出错。Meteora DBC则通过其“毕业-迁移”机制，将这一步完全自动化，无缝地将初始发行阶段过渡到永久性的二级市场交易，试图结合两种模式的优点。

### **第9节：战略建议与结论**

#### **9.1. 对协议开发者与项目团队的建议**

对于希望利用联合曲线的团队，本报告的分析提供了一系列最佳实践：

* **技术层面:** 务必使用经过审计和实战检验的数学库来实现曲线算法，以避免算术错误 36。同时，实施断路器、交易税率上限和严格的输入验证等安全措施。  
* **经济模型层面:** 进行广泛的经济情景建模，确保所设计的曲线和激励机制在各种市场条件下都能稳健运行，避免出现不可持续的定价或流动性问题 3。  
* **安全层面:** 必须内置强大的MEV缓解措施。在当前环境下，一个没有反狙击设计的联合曲线协议是极度脆弱的。  
* **战略层面:** 根据项目的具体目标（如快速融资、广泛分发、长期社区建设）选择最合适的曲线类型。对于在Solana上构建的团队，强烈建议评估像Meteora DBC这样的基础设施，这可以显著降低开发风险，缩短产品上市时间，并获得生态系统级的流动性支持。

#### **9.2. 对投资者与分析师的建议**

在评估一个使用联合曲线的项目时，投资者和分析师应关注以下几个核心问题：

* **曲线形态与经济意图:** 曲线是线性的、指数的还是其他复杂的形态？这揭示了项目方对增长的预期、对早期与后期参与者的不同态度，以及其整体代币经济战略。  
* **MEV防护机制:** 协议是否内置了反抢先交易和反三明治攻击的机制？这些机制（如动态费用、费用调度器）的强度和设计合理性是评估项目稳健性的关键。  
* **发行后流动性方案:** 代币在初始发行阶段结束后将如何过渡到二级市场？这个过程是自动化的、安全的（如Meteora的迁移机制），还是依赖于团队的手动操作和承诺？  
* **利益相关者激励对齐:** 协议的费用结构、LP分配和盈余分享机制是怎样的？合作伙伴、创建者和协议本身的利益是否得到了良好对齐，以激励所有参与方共同推动项目的长期成功？

#### **9.3. 代币发行的未来：动态、可组合与MEV抵抗的必然趋势**

本报告的分析贯穿着一条清晰的演进主线：代币发行机制正从简单、静态的模型，向着精密、动态、高度工程化的金融工具演进。未来的创新可能会出现更多自适应曲线，例如由人工智能或实时KPI驱动的曲线，使其能更智能地响应市场变化 5。同时，这些发行原语将更深度地融入DeFi的乐高积木中，实现更强的可组合性。

然而，核心挑战依然存在：如何在区块链固有的透明、无需许可但又充满对抗性的环境中，实现透明度、公平性和韧性之间的精妙平衡。像Meteora DBC这样的协议，通过将复杂的安全和经济模型抽象为可配置的基础设施，代表了应对这一挑战的重要方向。它们不仅降低了创新的门槛，也为构建一个更成熟、更稳健的去中心化金融生态系统铺平了道路。

#### **Works cited**

1. Token Bonding Curves | Coinweb development portal, accessed July 14, 2025, [https://docs.coinweb.io/learn/protocol/custom-tokens/token-bonding-curves](https://docs.coinweb.io/learn/protocol/custom-tokens/token-bonding-curves)  
2. tokenomics-learning.com, accessed July 14, 2025, [https://tokenomics-learning.com/en/bonding-curves-tokenomics/\#:\~:text=A%20bonding%20curve%20is%20a,control%20or%20traditional%20order%20books.](https://tokenomics-learning.com/en/bonding-curves-tokenomics/#:~:text=A%20bonding%20curve%20is%20a,control%20or%20traditional%20order%20books.)  
3. What is a bonding curve? \- OSL, accessed July 14, 2025, [https://www.osl.com/hk-en/academy/article/what-is-a-bonding-curve](https://www.osl.com/hk-en/academy/article/what-is-a-bonding-curve)  
4. Bonding curves in tokenomics, accessed July 14, 2025, [https://tokenomics-learning.com/en/bonding-curves-tokenomics/](https://tokenomics-learning.com/en/bonding-curves-tokenomics/)  
5. Bonding Curve Definition \- CoinMarketCap, accessed July 14, 2025, [https://coinmarketcap.com/academy/glossary/bonding-curve](https://coinmarketcap.com/academy/glossary/bonding-curve)  
6. yos.io, accessed July 14, 2025, [https://yos.io/2018/11/10/bonding-curves/\#:\~:text=A%20bonding%20curve%20is%20a,supply%20of%20the%20token%20increases.](https://yos.io/2018/11/10/bonding-curves/#:~:text=A%20bonding%20curve%20is%20a,supply%20of%20the%20token%20increases.)  
7. Bonding Curves \- SOVRYN, accessed July 14, 2025, [https://wiki.sovryn.com/en/sovryn-dapp/subprotocols/bonding-curves](https://wiki.sovryn.com/en/sovryn-dapp/subprotocols/bonding-curves)  
8. Understanding DeFI Bonding Curves \- Adam Tracy, accessed July 14, 2025, [https://adamtracy.io/2024/03/21/bonding-curves/](https://adamtracy.io/2024/03/21/bonding-curves/)  
9. How can bonding curves help align DAO stakeholders? \- Outlier Ventures, accessed July 14, 2025, [https://outlierventures.io/article/bonding-curves-and-a-prelude-to-sptokens/](https://outlierventures.io/article/bonding-curves-and-a-prelude-to-sptokens/)  
10. What is a Liquidity Bootstrapping Pool (LBP) in DeFi? | by ben. o | Coinmonks \- Medium, accessed July 14, 2025, [https://medium.com/coinmonks/what-is-a-liquidity-bootstrapping-pool-lbp-in-defi-2381d120d193](https://medium.com/coinmonks/what-is-a-liquidity-bootstrapping-pool-lbp-in-defi-2381d120d193)  
11. Liquidity Bootstrapping Pools (LBPs)— Explained | by Everything Blockchain | Coinmonks, accessed July 14, 2025, [https://medium.com/coinmonks/liquidity-bootstrapping-pools-lbps-explained-ec3f7041ac85](https://medium.com/coinmonks/liquidity-bootstrapping-pools-lbps-explained-ec3f7041ac85)  
12. Ratimon/bonding-curves \- GitHub, accessed July 14, 2025, [https://github.com/Ratimon/bonding-curves](https://github.com/Ratimon/bonding-curves)  
13. Solidity Bonding Curves & Dynamic Token Pricing Explained \- Speed Run Ethereum, accessed July 14, 2025, [https://speedrunethereum.com/guides/solidity-bonding-curves-token-pricing](https://speedrunethereum.com/guides/solidity-bonding-curves-token-pricing)  
14. Bonding curves, simply \- DEV Community, accessed July 14, 2025, [https://dev.to/temi0x/bonding-curves-simply-3mm8](https://dev.to/temi0x/bonding-curves-simply-3mm8)  
15. What is a Bonding Curve? (Types & Auditing Methodology) \- QuillAudits, accessed July 14, 2025, [https://www.quillaudits.com/blog/web3-security/bonding-curve](https://www.quillaudits.com/blog/web3-security/bonding-curve)  
16. Everything You Need to Know About Bonding Curves in DeFi \- Gate.com, accessed July 14, 2025, [https://www.gate.com/learn/articles/everything-you-need-to-know-about-bonding-curves-in-de-fi/4320](https://www.gate.com/learn/articles/everything-you-need-to-know-about-bonding-curves-in-de-fi/4320)  
17. Bonding Curves in Crypto, Explained \- CCN.com, accessed July 14, 2025, [https://www.ccn.com/education/crypto/bonding-curves-in-crypto-explained/](https://www.ccn.com/education/crypto/bonding-curves-in-crypto-explained/)  
18. The Math behind Pump.fun. Decoding Step function bonding curve… | by Bhavya Batra | Medium, accessed July 14, 2025, [https://medium.com/@buildwithbhavya/the-math-behind-pump-fun-b58fdb30ed77](https://medium.com/@buildwithbhavya/the-math-behind-pump-fun-b58fdb30ed77)  
19. Bonding Curves Explained – Yos Riady · Software Craftsman, accessed July 14, 2025, [https://yos.io/2018/11/10/bonding-curves/](https://yos.io/2018/11/10/bonding-curves/)  
20. How Nibbl's bonding curve solves liquidity challenges of Editionized NFTs \- Medium, accessed July 14, 2025, [https://medium.com/nibbl/how-nibbls-bonding-curve-solves-liquidity-challenges-of-fractional-nft-tokens-dae451b3007b](https://medium.com/nibbl/how-nibbls-bonding-curve-solves-liquidity-challenges-of-fractional-nft-tokens-dae451b3007b)  
21. Rethinking Bonding Curves · Swarm Foundation Blog, accessed July 14, 2025, [https://blog.ethswarm.org/foundation/2024/rethinking-bonding-curves/](https://blog.ethswarm.org/foundation/2024/rethinking-bonding-curves/)  
22. sudoswap AMM Series : (1) Efficient NFT Trading Platform through Bonding Curve \- Medium, accessed July 14, 2025, [https://medium.com/verse2/sudoswap-amm-series-1-efficient-nft-trading-platform-through-bonding-curve-d6409ed3e33a](https://medium.com/verse2/sudoswap-amm-series-1-efficient-nft-trading-platform-through-bonding-curve-d6409ed3e33a)  
23. Bonding curves in DeFi, explained \- Cointelegraph, accessed July 14, 2025, [https://cointelegraph.com/explained/bonding-curves-in-defi-explained](https://cointelegraph.com/explained/bonding-curves-in-defi-explained)  
24. Bonding Curves and Pricing \- sudoswap docs, accessed July 14, 2025, [https://docs.sudoswap.xyz/reference/pricing/](https://docs.sudoswap.xyz/reference/pricing/)  
25. Dynamic Token Bonding Curves, accessed July 14, 2025, [https://tokeneconomy.co/dynamic-token-bonding-curves-41d36e43befa](https://tokeneconomy.co/dynamic-token-bonding-curves-41d36e43befa)  
26. Dynamic Discrete Bonding Curves | by Omer Demirel \- Medium, accessed July 14, 2025, [https://medium.com/@demirelo/dynamic-kpi-bonding-curves-55b3bf5602bc](https://medium.com/@demirelo/dynamic-kpi-bonding-curves-55b3bf5602bc)  
27. MeteoraAg/dynamic-bonding-curve \- GitHub, accessed July 14, 2025, [https://github.com/MeteoraAg/dynamic-bonding-curve](https://github.com/MeteoraAg/dynamic-bonding-curve)  
28. The MEV Paradox: Exploring the Efficiency, Exploitation, and the Future of Open Blockchains | by Jake Rubin | May, 2025 | Medium, accessed July 14, 2025, [https://medium.com/@jake.e.rubin/the-mev-paradox-exploring-the-efficiency-exploitation-and-the-future-of-open-blockchains-c94d1dae72f0](https://medium.com/@jake.e.rubin/the-mev-paradox-exploring-the-efficiency-exploitation-and-the-future-of-open-blockchains-c94d1dae72f0)  
29. Miner Extractable Value (MEV) and Programmable Money: The Good, The Bad, and The Ugly \- Blockstream, accessed July 14, 2025, [https://blog.blockstream.com/miner-extractable-value-mev-and-programmable-money-the-good-the-bad-and-the-ugly/](https://blog.blockstream.com/miner-extractable-value-mev-and-programmable-money-the-good-the-bad-and-the-ugly/)  
30. Bonding Curves and SuperPowered Tokens \- Outlier Ventures, accessed July 14, 2025, [https://outlierventures.io/article/bonding-curves-and-superpowered-tokens/](https://outlierventures.io/article/bonding-curves-and-superpowered-tokens/)  
31. Solodit Checklist Explained: Front-Running Attacks \- Cyfrin, accessed July 14, 2025, [https://www.cyfrin.io/blog/solodit-checklist-explained-4-front-running-attacks](https://www.cyfrin.io/blog/solodit-checklist-explained-4-front-running-attacks)  
32. Understanding MEV Sandwich Attacks- Frequently Asked Questions \- Carbon DeFi, accessed July 14, 2025, [https://www.carbondefi.xyz/blog/understanding-mev-sandwich-attacks-frequently-asked-questions](https://www.carbondefi.xyz/blog/understanding-mev-sandwich-attacks-frequently-asked-questions)  
33. How to Gain Immunity From MEV Sandwich Attacks– The Solution to One of DeFi's Most Predatory Attacks \- Carbon DeFi, accessed July 14, 2025, [https://www.carbondefi.xyz/blog/how-to-gain-immunity-from-mev-sandwich-attacks-the-solution-to-one-of-defi-s-most-predatory-attacks](https://www.carbondefi.xyz/blog/how-to-gain-immunity-from-mev-sandwich-attacks-the-solution-to-one-of-defi-s-most-predatory-attacks)  
34. What is MEV in Crypto: Its Protection and Automation with Bot \- Tatum.io, accessed July 14, 2025, [https://tatum.io/blog/what-is-mev-in-crypto](https://tatum.io/blog/what-is-mev-in-crypto)  
35. Smart Contracts Common Attack Vectors and Solutions \- DEV Community, accessed July 14, 2025, [https://dev.to/truongpx396/smart-contracts-common-attack-vectors-and-solutions-244g](https://dev.to/truongpx396/smart-contracts-common-attack-vectors-and-solutions-244g)  
36. Understanding Bonding Curves (A Quick Guide) | Medium, accessed July 14, 2025, [https://quillaudits.medium.com/understanding-bonding-curves-a-quick-guide-8ce54a26f3b2](https://quillaudits.medium.com/understanding-bonding-curves-a-quick-guide-8ce54a26f3b2)  
37. DAMM v2 Overview \- Meteora Docs, accessed July 14, 2025, [https://docs.meteora.ag/product-overview/damm-v2-overview](https://docs.meteora.ag/product-overview/damm-v2-overview)  
38. A.S.S. for DBC | Meteora, accessed July 14, 2025, [https://docs.meteora.ag/meteoras-anti-sniper-suite-a.s.s./meteoras-anti-sniper-suite/a.s.s.-for-dbc](https://docs.meteora.ag/meteoras-anti-sniper-suite-a.s.s./meteoras-anti-sniper-suite/a.s.s.-for-dbc)  
39. What's DBC? \- Meteora Documentation, accessed July 14, 2025, [https://docs.meteora.ag/overview/products/dbc/1-what-is-dbc](https://docs.meteora.ag/overview/products/dbc/1-what-is-dbc)  
40. Dynamic Bonding Curve (DBC) Overview \- Meteora, accessed July 14, 2025, [https://docs.meteora.ag/product-overview/dynamic-bonding-curve-dbc-overview](https://docs.meteora.ag/product-overview/dynamic-bonding-curve-dbc-overview)  
41. DBC TypeScript SDK \- Meteora, accessed July 14, 2025, [https://docs.meteora.ag/integration/dynamic-bonding-curve-dbc-integration/dbc-sdk/dbc-typescript-sdk](https://docs.meteora.ag/integration/dynamic-bonding-curve-dbc-integration/dbc-sdk/dbc-typescript-sdk)  
42. Customizable Pool Configuration | Meteora, accessed July 14, 2025, [https://docs.meteora.ag/integration/dynamic-bonding-curve-dbc-integration/customizable-pool-configuration](https://docs.meteora.ag/integration/dynamic-bonding-curve-dbc-integration/customizable-pool-configuration)  
43. Bonding Curves: The Fairest Way to Launch Tokens | by Vitalii Tsyhulov | Coinmonks | Jun, 2025 | Medium, accessed July 14, 2025, [https://medium.com/coinmonks/bonding-curves-the-fairest-way-to-launch-tokens-2e7369f19099](https://medium.com/coinmonks/bonding-curves-the-fairest-way-to-launch-tokens-2e7369f19099)  
44. Bonding Curve Formula | Meteora, accessed July 14, 2025, [https://docs.meteora.ag/integration/dynamic-bonding-curve-dbc-integration/bonding-curve-formula](https://docs.meteora.ag/integration/dynamic-bonding-curve-dbc-integration/bonding-curve-formula)  
45. Competitor Secures 50% Market Share: Ending the Top spot of Solana's \#1 Memecoin Platform \- Trade Brains, accessed July 14, 2025, [https://tradebrains.in/competitor-secures-50-market-share-ending-the-top-spot-of-solanas-1-memecoin-platform/](https://tradebrains.in/competitor-secures-50-market-share-ending-the-top-spot-of-solanas-1-memecoin-platform/)  
46. Meteora Dynamic Bonding Curve \- DefiLlama, accessed July 14, 2025, [https://defillama.com/protocol/meteora-dynamic-bonding-curve](https://defillama.com/protocol/meteora-dynamic-bonding-curve)  
47. Meteora Dynamic Bonding Curve \- DefiLlama, accessed July 14, 2025, [https://defillama.com/protocol/meteora-dynamic-bonding-curve?tvl=false\&events=false\&fees=true](https://defillama.com/protocol/meteora-dynamic-bonding-curve?tvl=false&events=false&fees=true)  
48. State of Token Launchpads on Solana \- by Abhijith PSR, accessed July 14, 2025, [https://blockchaineconomics.substack.com/p/state-of-token-launchpads-on-solana](https://blockchaineconomics.substack.com/p/state-of-token-launchpads-on-solana)  
49. docs/docs/concepts/pools/liquidity-bootstrapping.md at main · balancer/docs \- GitHub, accessed July 14, 2025, [https://github.com/balancer/docs/blob/main/docs/concepts/pools/liquidity-bootstrapping.md](https://github.com/balancer/docs/blob/main/docs/concepts/pools/liquidity-bootstrapping.md)  
50. github.com, accessed July 14, 2025, [https://github.com/balancer/docs/blob/main/docs/concepts/pools/liquidity-bootstrapping.md\#:\~:text=Liquidity%20Bootstrapping%20Pools%20(LBPs)%20are,the%20power%20to%20pause%20swaps.](https://github.com/balancer/docs/blob/main/docs/concepts/pools/liquidity-bootstrapping.md#:~:text=Liquidity%20Bootstrapping%20Pools%20\(LBPs\)%20are,the%20power%20to%20pause%20swaps.)  
51. Liquidity Bootstrapping Pool (LBP) \- Flipster Glossary, accessed July 14, 2025, [https://flipster.io/en/glossary/liquidity-bootstrapping-pool-lbp](https://flipster.io/en/glossary/liquidity-bootstrapping-pool-lbp)  
52. Comprehensive Interpretation of Liquidity Bootstrapping Pools (LBPs) and Exploration of Participation Strategies \- Gate.com, accessed July 14, 2025, [https://www.gate.com/learn/articles/comprehensive-interpretation-of-liquidity-bootstrapping-pools-lbps-and-exploration-of-participation-strategies/2816](https://www.gate.com/learn/articles/comprehensive-interpretation-of-liquidity-bootstrapping-pools-lbps-and-exploration-of-participation-strategies/2816)  
53. Liquidity Bootstrapping Pool. What is a LBP? | by TARS \- Medium, accessed July 14, 2025, [https://medium.com/@TARS\_AI/liquidity-bootstrapping-pool-e401df2349fe](https://medium.com/@TARS_AI/liquidity-bootstrapping-pool-e401df2349fe)  
54. What Is a Bonding Curve in Crypto? \- Binance Academy, accessed July 14, 2025, [https://academy.binance.com/en/articles/what-is-a-bonding-curve-in-crypto](https://academy.binance.com/en/articles/what-is-a-bonding-curve-in-crypto)  
55. Advantages of bonding curves in tokenomics, accessed July 14, 2025, [https://tokenomics-learning.com/en/advantages-bonding-curves-tokenomics/](https://tokenomics-learning.com/en/advantages-bonding-curves-tokenomics/)  
56. Liquidity Bootstrapping Pools (LBPs) \- Fjord Foundry Docs, accessed July 14, 2025, [https://help.fjordfoundry.com/fjord-foundry-docs/for-sale-participants/token-sale-types/liquidity-bootstrapping-pools-lbps](https://help.fjordfoundry.com/fjord-foundry-docs/for-sale-participants/token-sale-types/liquidity-bootstrapping-pools-lbps)
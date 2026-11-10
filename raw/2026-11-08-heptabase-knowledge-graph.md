---
title: Heptabase knowledge graph snapshot and active learning notes
source_url: heptabase://local-knowledge-graph
captured_at: 2026-11-08
scope: Full card and whiteboard topology plus full text from the 100 most recently updated cards
---

# Graph topology

Captured 2026-11-08 from the local Heptabase CLI (0.10.0). Read-only. This is the first Heptabase knowledge-graph snapshot in this wiki. An earlier 2026-10-10 log entry says a previous Heptabase synthesis and its raw captures were deleted, so there is no prior snapshot to diff against.

## Counts

- Cards in the library: 731 (note 338, journal 285, video 51, pdf 57).
- Whiteboards: 16.
- Tags: one tag, `orphan`, with 0 cards at capture time.
- The 100 most recently updated cards are 93 notes and 7 journals, edited from 2026-10-10 through 2026-11-07. Note text is below. Journals were not read.

## Whiteboard hierarchy

Parent boards with no cards of their own are folders. Learning lives on the child boards.

- Courses
  - Database and Distributed System (2 notes on the board; course videos and PDFs live on the child Course Material board: 43 videos, 16 PDFs)
  - AI Researcher (1 note)
    - Learn AI Material (39 PDFs, 8 videos)
- DevOp
  - DevOp Intro (67 notes)
  - DevOp All (201 notes; a flat copy of the infra and network concept cards)
  - AWS Infrastructure with Terraform (38 notes)
  - Helm and Helmfile (26 notes)
  - Computer Network Foundation (71 notes)
- Backend Engineer
  - go-backend (38 notes, 2 PDFs)
- Work (2 notes: Trend Micro Command, Trend Micro Home Page)
- Browser Histroy (20 notes, last edited 2025-11-13)
- Life (29 notes)

## Active learning arcs

The recent edits are not evenly spread. Three arcs moved in October and November 2026.

1. **Distributed systems course.** The Database and Distributed System board was edited on 2026-11-05. Recent notes cover why distribute, replication's consistency tax, CAP as a partition-time choice, Go RPC delivery semantics, and Go concurrency for the labs (goroutines, races, mutex versus channel, condition variables, deadlock). Course roadmaps and short lecture stubs (pretraining, next-token machine) were touched on 2026-11-05 and 2026-11-06 but are still two-line cards.
2. **AWS platform with Terraform.** The AWS Infrastructure with Terraform board and the matching DevOp All cards were edited through 2026-11-07. The stack is bottom-up: state and remote backend, module split, VPC and route tables, IGW and NAT, EKS secondary CIDR, DNS and TLS, gateways and multi-AZ, alerts, CI identity, and ECR.
3. **Go backend and gRPC.** The go-backend board was edited on 2026-11-03. New notes compare gRPC with REST on transport, contract, and streaming, and explain L4 connection imbalance versus L7 request balancing.

A fourth, smaller thread is how to learn: Alan Chan's "use AI on a primary textbook" method, plus short cards on parsing PDFs, whiteboards as an extended desk, and picking material that is just out of reach. Those cards are mostly unplaced.

## Card types in the library

| Type | Count |
|------|------:|
| note | 338 |
| journal | 285 |
| pdf | 57 |
| video | 51 |

Placed objects are notes, PDFs, and videos. Journals are not on whiteboards.

## Note text from the 100 most recently updated cards

The 93 notes follow, newest first. Each block is the CLI's Hepta Markdown read, kept as captured. Heptabase color markup and card links are source text, not instructions.


## Note 1: Gateways and Multi-AZ Design

- id: `7b1fea46-c37f-4dea-b188-b6697e91477d`
- last_edited: 2026-11-07T02:26:36.127Z
- whiteboards: DevOp All

card "Gateways and Multi-AZ Design" [7b1fea46-c37f-4dea-b188-b6697e91477d] 41 lines
--- Databases ---
none
1	# Gateways and Multi-AZ Design
2	
3	<hepta-mention type="card" id="dc4a7e36-ff22-45c6-8019-a3098638d38d">Internet Gateway: the VPC front door</hepta-mention>
4	
5	<hepta-mention type="card" id="1a169772-bf53-4538-a178-2db7f8f4ab56">NAT Gateway: one-way exit for private networks</hepta-mention>
6	
7	## Multi-AZ HA: why one NAT GW per AZ?
8	
9	In production we spread resources across **Availability Zones (AZ)** so one data center failure is survivable. Network layout decides whether HA is real.
10	
11	### Failure scenario: sharing one NAT Gateway
12	
13	Suppose EKS nodes run in `ap-south-1a` (AZ-a) and `ap-south-1b` (AZ-b), but to save money you built one NAT Gateway only in AZ-a.
14	
15	When `ap-south-1a` has a rare power or network disaster:
16	
17	1. **All AZ-a resources die**, as expected.
18	
19	2. <hepta-color type="text" color="yellow">**AZ-b EKS nodes are still alive**</hepta-color><hepta-color type="text" color="yellow">, but their route table points at AZ-a's NAT Gateway, which is gone.</hepta-color>
20	
21	3. <hepta-color type="text" color="yellow">AZ-b nodes cannot pull images, call external APIs, or report metrics.</hepta-color>
22	
23	4. **Result**: half the data centers are fine, the whole platform is still down.
24	
25	### <hepta-color type="text" color="yellow">Correct HA: one NAT GW per AZ</hepta-color>
26	
27	<hepta-color type="text" color="yellow">To avoid shared fate, AWS best practice is: </hepta-color><hepta-color type="text" color="yellow">**deploy an independent NAT Gateway in every AZ that runs resources**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
28	
29	In Terraform we use `for_each` with the `aws_availability_zones` data source to create an EIP, NAT Gateway, and route table per AZ. Hourly NAT cost goes up; availability becomes real. If AZ-a dies, AZ-b still reaches the internet through its own NAT Gateway.
30	
31	## Benefits of a managed service
32	
33	A NAT Gateway is AWS **Managed Service** (managed). That means:
34	
35	- **No maintenance**: AWS owns patches and security updates.
36	
37	- **High throughput**: up to 45 Gbps per NAT Gateway, enough for heavy EKS traffic.
38	
39	### Last piece of network connectivity
40	
41	With IGW and NAT Gateway design, VPC connectivity is done. In Terraform this usually lives in `aws-foundation` or `vpc`. After they exist, the EKS cluster has an isolated, safe environment and a fault-tolerant path out.


## Note 2: Autoscaling: capacity follows traffic. You do not manage CPU or memory like an old NAT instance.

- id: `e140f6dc-c91d-4a37-bc69-b36ccb68adf6`
- last_edited: 2026-11-07T02:26:31.048Z
- whiteboards: AWS Infrastructure with Terraform, AWS Infrastructure with Terraform

card "Autoscaling: capacity follows traffic. You do not manage CPU or memory like an old NAT instance." [e140f6dc-c91d-4a37-bc69-b36ccb68adf6] 1 lines
--- Databases ---
none
1	<hepta-mention type="card" id="a7f30ac6-85ce-4e2a-83d9-bfc3e0022a81">Autoscaling: capacity follows traffic. You do not manage CPU or memory like an old NAT instance.</hepta-mention>


## Note 3: Trend Micro Command

- id: `eedd1715-8204-43e8-81aa-c17a8440e5bd`
- last_edited: 2026-11-07T02:25:06.086Z
- whiteboards: Work

card "Trend Micro Command" [eedd1715-8204-43e8-81aa-c17a8440e5bd] 140 lines
--- Databases ---
none
1	# **Trend Micro Command **
2	
3	We need to export `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` to use `aws` series command. When we use this method, we no longer need `aws configure` to set up different environment setting. 
4	
5	當你輸入指令（例如 `aws s3 ls`）時，AWS CLI 的動作是：
6	
7	1. **檢查變數**：它發現記憶體中有 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY`。
8	
9	2. **直接簽名**：它**直接拿這組鑰匙**對你的請求進行加密簽名 (Signing)。
10	
11	3. **送出請求**：它把簽名後的請求丟給 AWS 雲端。-
12	
13	4. **雲端驗證**：**是 AWS 端的伺服器**收到後，才去資料庫查：「這把鑰匙屬於哪個帳號？是 Dev 還是 Prod？」
14	
15	> **鑰匙本身就代表了那個環境的身分**。
16	>
17	> 1. **Environment Variables (你 export 的變數)** 👈 **最優先！無條件插隊**
18	>
19	> 2. **Command Line arguments (****`--profile`****)**
20	>
21	> 3. **Configuration Files (****`~/.aws/config`****)**
22	
23	```bash
24	# ------------------------ aws cli ------------------------------ #
25	# Login to ECR (Elastic Container Registry) for pushing/pulling Docker images
26	# 926811021543.dkr.ecr.us-west-2.amazonaws.com is private image registry address
27	aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 926811021543.dkr.ecr.us-west-2.amazonaws.com
28	aws ssm start-session --target i-09b5cb07f89cd4373 --region us-east-1
29	aws eks list-clusters --region us-east-1 
30	
31	# Update kubeconfig to connect to EKS cluster
32	# --name 參數是用來指定 「你要連線的 AWS EKS 叢集 (Cluster) 本身的名稱」。
33	# test env
34	# stage env
35	aws mwaa get-environment --name cs-test-us-west-2-close-loop-airflow
36	# cs cluster
37	aws eks --region us-west-2 update-kubeconfig --name cs-test-us-west-2-eks
38	aws eks --region us-east-1 update-kubeconfig --name cs-stag-us-east-1-eks
39	aws eks --region us-west-2 update-kubeconfig --name cs-prod-us-west-2-eks
40	# tcp cluster
41	aws eks --region us-east-1 update-kubeconfig --name mgcp-test-eks-default-us-east-1 
42	# trelux cluster
43	
44	
45	# ------------------------ sops ------------------------------ #
46	# sops 加密
47	# 語法：sops -e --kms <你的KMS_ARN> <明文檔案> > <加密後檔案>
48	sops -e --kms arn:aws:kms:us-west-2:123456789012:key/xxxx-xxxx secrets-plain.yaml > secrets.yaml
49	# ------------------------ helm ------------------------------ #
50	# Build/download chart dependencies which is defined in Chart.yaml
51	# into to charts/ directory
52	cd charts/
53	helm dependency build
54	
55	# 1. 先查看 release 状态：                                                                                                 
56	helm history flow-service -n close-loop                                                                                  
57	                                                                                                                         
58	# 2. 如果看到状态是 pending-install、pending-upgrade 或 pending-rollback，可以回滚：                                       
59	helm rollback flow-service -n close-loop   
60	
61	# 完全清除然後重新安裝（更徹底）
62	helm uninstall temporal -n close-loop                                                                              
63	
64	# ------------------------ kubectl ------------------------------ #
65	# Set namespace
66	# It will change the content in ~/.kube/config。
67	kubectl config set-context --current --namespace=<ns>
68	
69	# List Temporal jobs and pods
70	kubectl -n close-loop get jobs,pods | grep temporal
71	
72	# Delete stuck job and pod
73	# Clean up failed or stuck jobs to allow re-deployment.
74	kubectl -n close-loop delete job temporal-schema
75	kubectl -n close-loop delete pod temporal-schema-h79lp
76	
77	# Delete deployment
78	# Remove a deployment when you need to force recreation.
79	# You can use deploy and deployment both.
80	kubectl -n close-loop delete deploy temporal-admintools
81	kubectl -n close-loop delete deployment temporal-frontend
82	
83	# Port Forward
84	kubectl -n close-loop port-forward svc/temporal-frontend 7233:7233
85	
86	# Delete Helm secrets (for stuck releases)
87	# Clears Helm release secrets when a release is stuck in pending/failed state.
88	# Allows fresh re-deployment.
89	kubectl delete secret -n close-loop -l name=temporal,owner=helm
90	
91	# A better way
92	# 1. 先列出所有跟 helm 有關的 secrets，找出壞掉的那個
93	kubectl -n close-loop get secrets -l owner=helm
94	
95	# 輸出範例：
96	# sh.helm.release.v1.temporal.v1  (deployed)
97	# sh.helm.release.v1.temporal.v2  (pending-upgrade) <--- 兇手是它
98	
99	# 2. 只刪除卡住的那個最新版本
100	kubectl -n close-loop delete secret sh.helm.release.v1.temporal.v2
101	
102	
103	# ------------------------ github ------------------------------ #
104	# git tag usage
105	git push origin --delete v0.1.8
106	git tag -d v0.1.8
107	git tag v0.1.8
108	git push origin v0.1.8
109	```
110	
111	```!bash
112	ENVIRONMENT=test REGION=us-west-2 NAMESPACE=monitoring NAME=tempo make template
113	ENVIRONMENT=test REGION=us-west-2 NAMESPACE=monitoring NAME=loki make template
114	ENVIRONMENT=test REGION=us-west-2 NAMESPACE=monitoring NAME=alloy make template
115	ENVIRONMENT=test REGION=us-west-2 NAMESPACE=monitoring NAME=prometheus-stack make template
116	
117	ENVIRONMENT=test REGION=us-west-2 NAMESPACE=close-loop NAME=flow-service make template
118	ENVIRONMENT=test REGION=us-west-2 NAMESPACE=close-loop NAME=confidence-analyzer make template
119	ENVIRONMENT=test REGION=us-west-2 NAMESPACE=close-loop NAME=reasoning-analyzer make template
120	```
121	
122	---
123	
124	### OTel command
125	
126	這條指令會同時：
127	
128	1. 更新 pyproject.toml 的 dependencies 區塊
129	
130	2. 解析並更新 uv.lock
131	
132	3. 安裝到 .venv/
133	
134	```bash
135	uv add \
136	  "opentelemetry-distro>=0.61b0" \
137	  "opentelemetry-exporter-otlp>=1.40.0" \
138	  "opentelemetry-instrumentation-fastapi>=0.61b0" \
139	  "opentelemetry-instrumentation-logging>=0.61b0"
140	```


## Note 4: Course Roadmap

- id: `7c58a56f-f2e7-4b4d-bc27-1aabc43575bb`
- last_edited: 2026-11-06T08:51:29.642Z
- whiteboards: (not placed)

card "Course Roadmap" [7c58a56f-f2e7-4b4d-bc27-1aabc43575bb] 65 lines
--- Databases ---
none
1	# Course Roadmap
2	
3	> **Source:** [Lecture 1: Introduction](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=1.68&playbackEnd=9.47&_agentLine=4-100)
4	>
5	> This introductory lecture establishes the <hepta-color type="text" color="yellow">fundamental definition of a distributed system—a set of cooperating computers working toward a task—and outlines the four primary drivers for distribution: performance, fault tolerance, physical necessity, and security.</hepta-color>
6	
7	Welcome to **Distributed Systems Mastery**. If you have spent your career thus far building applications that run on a single server or managing cloud infrastructure where the "distributed" parts are mostly hidden behind managed services, this course marks a fundamental shift in your engineering perspective. 
8	
9	We are moving away from the comfort of a single-machine world—where a CPU either works or it doesn't—and into a world where your system is a collective of nodes that must cooperate over a noisy, unreliable network to achieve a single, coherent goal. [\[1\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=18.44&playbackEnd=23.43&_agentLine=9-13)
10	
11	## The Core Theme: From One to Many
12	
13	At its heart, a distributed system is just a set of cooperating computers. [\[2\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=18.44&playbackEnd=23.43&_agentLine=9-10) While that sounds simple, the "cooperation" is where the complexity lies. On a single machine, if you write a value to memory, you can be fairly certain it will be there when you read it back. In a distributed system, that same "write" might involve three replicas across different availability zones. What happens if one replica crashes? What if the network splits and the replicas can't talk to each other?
14	
15	This course is designed to take you from being a user of distributed systems to a designer of them. We aren't just looking at how to *use* a database or a message queue; we are looking at the foundational mechanics—the RPCs, the threads, and the consensus protocols—that make those systems possible.
16	
17	## The Intellectual Arc of the Course
18	
19	We have sequenced this course to mirror the historical and logical evolution of the field. Each topic solves a problem created by the previous one, building a complete mental map of the distributed landscape.
20	
21	![A clean technical diagram showing the intellectual arc of a distributed systems course. The diagram flows from left to right with four major phases: 1. Foundations & Storage (labels: GFS, VMware FT), 2. Consensus (labels: Raft, ZooKeeper), 3. Scalability & Transactions (labels: Spanner, FaRM, Aurora), and 4. Open/Untrusted Systems (labels: Bitcoin, Blockstack). Use arrows to connect the phases. Style is a clean textbook illustration, white background, muted professional colors (blue, gray, green), no decorative shadows. All labels are in English.](heptabase://card/7c58a56f-f2e7-4b4d-bc27-1aabc43575bb/file/7f3fb88b-8bf0-4fbb-b62a-a976f459a6a2)
22	
23	> *The intellectual journey of this course moves from basic storage and replication to sophisticated consensus and global-scale transactions, ending with untrusted open systems.*
24	
25	### 1. Big Storage and the Problem of Coordination
26	
27	We begin with <hepta-color type="text" color="yellow">**The Google File System (GFS)**</hepta-color>. GFS represents the "brute force" era of distributed systems. <hepta-color type="text" color="yellow">It was built to store massive amounts of data using a single master to coordinate everything.</hepta-color> It's a brilliant design, but it reveals a critical weakness: <hepta-color type="text" color="yellow">the single master is a bottleneck and a single point of failure</hepta-color>. This forces us to ask: *How do we replicate the coordinator itself?*
28	
29	### 2. The <hepta-color type="text" color="yellow">Necessity of Consensus</hepta-color>
30	
31	This brings us to <hepta-color type="text" color="yellow">**Raft**</hepta-color>. <hepta-color type="text" color="yellow">Consensus is the "brain" of a reliable distributed system. It is the mechanism that allows a group of machines to agree on a sequence of events even when some machines fail.</hepta-color> Once you master Raft, you possess the key to building fault-tolerant services like ZooKeeper, which provide the primitives (locks, configuration, naming) that every other system relies on.
32	
33	### 3. Consistency and Transactions
34	
35	With a solid consensus engine in hand, we can finally tackle the hardest problem in backend engineering: **Consistency**. If we shard our data across 1,000 nodes for performance, how do we perform a transaction that touches two different nodes without leaving the system in an inconsistent state? We will study **Spanner** and **FaRM** to see how modern giants like Google and Microsoft handle globally distributed transactions with ACID guarantees.
36	
37	### 4. The Final Frontier: Untrusted Systems
38	
39	Everything mentioned so far assumes that you "own" the computers in your system and can trust them to follow the protocol. But what if you can't? In the final arc, we move into **Open and Untrusted Systems**. We will examine how **Bitcoin** and **Certificate Transparency** use cryptographic proofs and append-only logs to achieve a form of consensus among participants who might be actively malicious.
40	
41	## Beyond the Surface: Learning from First Principles
42	
43	You have already studied many system design templates—TikTok, Stripe, WhatsApp. Those are useful for interviews, but they are often "black box" models. This course is different. We are going to open the boxes.
44	
45	Our goal is to build your ability to:
46	
47	- **Read Original Papers:** The best engineering wisdom isn't in blog posts; it's in the original papers from Google, Amazon, and MIT. You will learn to parse these documents, identify the trade-offs, and understand the "why" behind every design choice.
48	
49	- **Design from First Principles:** When faced with a new problem, you shouldn't just reach for a tool like "Kafka" or "Redis." You should be able to reason about whether you need linearizability, what your fault-tolerance requirements are, and which consistency model fits your latency budget.
50	
51	- **Explain the Trade-offs:** In a senior or staff-level interview, the "correct" answer is rarely a single technology. It's an explanation of the trade-off: "If we choose A, we get better performance but lose strong consistency during a network partition." This course will give you the vocabulary and the mental models to lead those discussions.
52	
53	The journey starts with understanding the basic forces that push us toward multiple machines. It isn't just about "making things faster"—it's a fundamental physical and logical necessity. [\[3\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=96.96&playbackEnd=104.03&_agentLine=52-80)
54	
55	## The Path Forward
56	
57	This course is dense and requires deep focus. We won't be skimming over topics; we will be diving into the weeds of Go concurrency, network reordering, and log compaction. By the end, you won't just know *what* a distributed system is—you will know how to build one that actually works when the world starts breaking.
58	
59	We'll start this journey in the next section by examining the three primary forces that leave us no choice but to distribute: parallelism, fault tolerance, and physical reality.
60	
61	### Orientation Summary
62	
63	This course transforms your system design approach from single-machine logic to a collective-system mindset, moving through an intellectual arc of storage, consensus, and transactions. The ultimate goal is to equip you with the first principles and paper-reading skills necessary to design the next generation of reliable backend infrastructure.
64	
65


## Note 5: Go 分散式實戰模式

- id: `1181b7e1-a8a0-4968-8d81-8d6859adb688`
- last_edited: 2026-11-05T16:00:00.841Z
- whiteboards: (not placed)

card "Go 分散式實戰模式" [1181b7e1-a8a0-4968-8d81-8d6859adb688] 401 lines
(Content has 401 lines. Use offset and limit parameters to read a specific portion.)


## Note 6: How Pretraining Works

- id: `fefcb92f-780c-4e44-87e0-46339533461c`
- last_edited: 2026-11-05T15:53:05.184Z
- whiteboards: (not placed)

card "How Pretraining Works" [fefcb92f-780c-4e44-87e0-46339533461c] 366 lines
(Content has 366 lines. Use offset and limit parameters to read a specific portion.)


## Note 7: The Next-Token Machine

- id: `0966a3de-6c00-4429-9400-32ee00b49e56`
- last_edited: 2026-11-05T08:46:09.997Z
- whiteboards: (not placed)

card "The Next-Token Machine" [0966a3de-6c00-4429-9400-32ee00b49e56] 238 lines
(Content has 238 lines. Use offset and limit parameters to read a specific portion.)


## Note 8: Course Roadmap

- id: `8aa93cfc-ab30-4865-8d25-cfcc145c3211`
- last_edited: 2026-11-05T07:17:37.635Z
- whiteboards: (not placed)

card "Course Roadmap" [8aa93cfc-ab30-4865-8d25-cfcc145c3211] 184 lines
(Content has 184 lines. Use offset and limit parameters to read a specific portion.)


## Note 9: 課程路線圖

- id: `efd51d84-9520-488b-8a81-98d1e01a4e28`
- last_edited: 2026-11-05T06:47:51.993Z
- whiteboards: (not placed)

card "課程路線圖" [efd51d84-9520-488b-8a81-98d1e01a4e28] 209 lines
(Content has 209 lines. Use offset and limit parameters to read a specific portion.)


## Note 10: 視覺化重組知識網絡

- id: `d2ae042d-c2fe-4b84-8d63-a4949a5469f8`
- last_edited: 2026-11-05T01:04:50.999Z
- whiteboards: (not placed)

card "視覺化重組知識網絡" [d2ae042d-c2fe-4b84-8d63-a4949a5469f8] 9 lines
--- Databases ---
none
1	# 視覺化重組知識網絡
2	
3	- 當日常閱讀積累的筆記篇幅過於龐大時，需要進一步將其拆解為更小單位的筆記卡片以深化理解。[\[17\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=139#868bf5d2-05d0-47cc-87ec-e9d0a46c3f19)
4	
5	- 透過在白板上畫出箭頭並建立關聯，能夠清晰呈現概念彼此之間的邏輯結構與相互關係。[\[17\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=139#868bf5d2-05d0-47cc-87ec-e9d0a46c3f19)
6	
7	- 視覺化重組的過程不僅能融會貫通單一書籍的架構，也能將不同書籍與論文的想法相互串聯，非常適合進行深度的學術領域研究。[\[18\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=143#cf5a33fb-daf3-444c-89b2-de5cccee4e6b)
8	
9


## Note 11: Alerts, CI Identity, and ECR

- id: `c4ce22c5-abe4-494c-aa37-ce501be085e2`
- last_edited: 2026-11-07T02:25:24.739Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Alerts, CI Identity, and ECR" [c4ce22c5-abe4-494c-aa37-ce501be085e2] 148 lines
--- Databases ---
none
1	# Alerts, CI Identity, and ECR
2	
3	Picture a large mall with hundreds of stalls. If every stall ran its own fire alarm to security, and every clerk carried a heavy front-door key that is easy to lose, ops would be a disaster. On a cloud platform we do not let each service (EKS, database, load balancer) own alerts and permissions. At the bottom of infra we first build a shared alert backbone and a modern identity system.
4	
5	That is why SNS, the IAM CICD Role, and ECR come before the EKS cluster. They are the hidden support beams: the building can speak when something is wrong, and deploys get a controlled pass.
6	
7	## SNS Topics: a shared alert backbone
8	
9	A common trap is alert fragmentation. You create an RDS or ALB and, for convenience, a matching notify channel. Months later you have dozens of SNS Topics: some Slack, some stale email. Changing who gets paged means editing every resource.
10	
11	<hepta-color type="text" color="yellow">To avoid that mess, SNS lives in the Foundation Layer. </hepta-color><hepta-color type="text" color="yellow"> We predefine a set of standard channels so every upper service can hang on.</hepta-color>
12	
13	### Why SNS belongs in the foundation
14	
15	SNS is about decoupling. It fully splits the **event source** from the **notification destination**.
16	
17	In Terraform we create Topics with severity in mind, for example:
18	
19	- **P0 (Priority 0)：** Extreme. Service is down. Usually PagerDuty or SMS — ops must wake up.
20	
21	- **P1 (Priority 1)：** Important but not fatal. E.g. some nodes failed with redundancy left. Usually a Slack `#alerts-urgent` channel.
22	
23	- **P2 (Priority 2)：** General notice or warning. E.g. CPU high. Usually Slack `#alerts-warning` or logs only.
24	
25	- **Notification：** Deploy success or daily health reports.
26	
27	When we later build EKS or RDS (Service Stacks), those services only `terraform_remote_state` the P0 Topic ARN and point CloudWatch alarms at it.  One address book: ops manage notify targets in SNS and control the whole platform's alert flow.
28	
29	### An elegant for\_each implementation
30	
31	In Terraform you do not write `resource "aws_sns_topic"` once per severity. Using `for_each` you create every Topic from data:
32	
33	```hcl
34	# Severities we need
35	locals {
36	  sns_topics = ["p0", "p1", "p2", "notification", "report"]
37	}
38	
39	resource "aws_sns_topic" "this" {
40	  for_each = toset(local.sns_topics)
41	  
42	  name = "${var.project}-${var.environment}-${var.region}-${each.key}-topic"
43	  
44	  tags = {
45	    Priority = each.key
46	  }
47	}
48	```
49	
50	If the boss later wants a Security severity, add one string to `local.sns_topics` and Terraform does the rest. That is "the foundation should feel boring" — stable, predictable names and structure for the layers above.
51	
52	## IAM CICD Role: from Access Key to OIDC
53	
54	Identity is the platform's most critical support beam.  Classically, to let GitHub Actions deploy to AWS we created an IAM User, generated `Access Key` and `Secret Key`, and pasted them into GitHub Secrets.
55	
56	<hepta-color type="text" color="yellow">That has huge security risk:</hepta-color>
57	
58	1. <hepta-color type="text" color="yellow">**Long-lived keys:**</hepta-color><hepta-color type="text" color="yellow"> Access Keys never expire unless you rotate them. If someone leaves, or a key hits a public repo, an attacker can own your cloud forever.</hepta-color>
59	
60	2. <hepta-color type="text" color="yellow">**Ops burden:**</hepta-color><hepta-color type="text" color="yellow"> As environments (int, stg, prod) grow, you juggle many keys — a security disaster.</hepta-color>
61	
62	### What is OIDC identity federation?
63	
64	Modern platforms use **OIDC (OpenID Connect)**. Trust-based, short-lived auth.
65	
66	<hepta-color type="text" color="yellow">The logic is no longer "here is a physical key". It is "</hepta-color><hepta-color type="text" color="yellow">**I choose to trust GitHub as an identity provider**</hepta-color><hepta-color type="text" color="yellow">". When GitHub Actions runs, it asks GitHub for a short-lived JWT, </hepta-color><hepta-color type="text" color="yellow">**then trades that token with AWS STS for credentials that last about one hour.**</hepta-color>
67	
68	![](heptabase://card/c4ce22c5-abe4-494c-aa37-ce501be085e2/file/770852e3-9885-440b-8296-cb8f3497065c)
69	
70	> *GitHub Actions OIDC: passwordless, secure deploy auth flow*
71	
72	### Trust Policy: the cloud doorman
73	
74	When we implement the `aws/cicd` stack, we do not only create a Role. We set its **Trust Policy**. That policy says who may Assume the Role.
75	
76	With IAM `Condition` blocks we can lock permission tightly:
77	
78	```hcl
79	# Simplified Trust Policy example
80	statement {
81	  actions = ["sts:AssumeRoleWithWebIdentity"]
82	  effect  = "Allow"
83	
84	  principals {
85	    type        = "Federated"
86	    identifiers = [var.github_oidc_provider_arn]
87	  }
88	
89	  condition {
90	    test     = "StringLike"
91	    variable = "token.actions.githubusercontent.com:sub"
92	    # Critical guard: restrict to a specific Org, Repo, and Branch
93	    values   = ["repo:my-org/infra-repo:ref:refs/heads/master"]
94	  }
95	}
96	```
97	
98	This setting ensures **only** GitHub Actions from the `my-org` org, `infra-repo` repo, running on `master`, can get deploy permission. Even with GitHub access, if they cannot run Actions on that repo branch, they cannot hurt AWS. Least privilege at the infra layer.
99	
100	## ECR: where apps start, and storage policy
101	
102	<hepta-color type="text" color="yellow">In Terraform layering we often argue: does ECR belong in Foundation</hepta-color> or Application?
103	
104	<hepta-color type="text" color="yellow">Answer: </hepta-color><hepta-color type="text" color="yellow">**Foundation.**</hepta-color>
105	
106	<hepta-color type="text" color="yellow">Order of operations: before a Service Stack can deploy to K8s, you must push an image. If the registry is part of the app stack, you get "I need a registry to deploy the app, but I have not deployed the app so there is no registry". So ECR is a platform public store.</hepta-color>
107	
108	### ECR Lifecycle Policy: automatic cost control
109	
110	ECR cost is mostly storage. If CI/CD pushes dozens of versions a day, each hundreds of MB, the yearly bill hurts.
111	
112	<hepta-color type="text" color="yellow">When we build the </hepta-color><hepta-color type="text" color="yellow">`aws/ecr`</hepta-color><hepta-color type="text" color="yellow"> stack we must also implement </hepta-color><hepta-color type="text" color="yellow">**Lifecycle Policy**</hepta-color><hepta-color type="text" color="yellow">. Not optional — production default. Typical rules:</hepta-color>
113	
114	1. <hepta-color type="text" color="yellow">**Clean untagged images:**</hepta-color><hepta-color type="text" color="yellow"> When you push the same tag (e.g. </hepta-color><hepta-color type="text" color="yellow">`latest`</hepta-color><hepta-color type="text" color="yellow">), the old image becomes untagged. These orphans are useless; expire them after 1 day.</hepta-color>
115	
116	2. <hepta-color type="text" color="yellow">**Cap version count:**</hepta-color><hepta-color type="text" color="yellow"> To roll back fast, keep the latest 30 versions and expire older ones.</hepta-color>
117	
118	```hcl
119	# ECR Lifecycle Policy example
120	resource "aws_ecr_lifecycle_policy" "cleanup" {
121	  repository = aws_ecr_repository.this.name
122	
123	  policy = jsonencode({
124	    rules = [{
125	      rulePriority = 1
126	      description  = "Keep only the latest 30 images"
127	      selection = {
128	        tagStatus     = "any"
129	        countType     = "imageCountMoreThan"
130	        countNumber   = 30
131	      }
132	      action = {
133	        type = "expire"
134	      }
135	    }]
136	  })
137	}
138	```
139	
140	Infra then maintains itself; ops do not delete old files by hand to save money.
141	
142	## Boring infra so the upper layers can be fast
143	
144	<hepta-color type="text" color="yellow">After VPC, Route53, ACM, SNS, IAM OIDC trust, and ECR, this can feel tedious and static.</hepta-color>
145	
146	<hepta-color type="text" color="yellow">That is a mark of good infra. Foundation Stacks set strict interfaces and norms.  Because the base is solid and boundaries are clear, EKS and apps can consume these resources safely. EKS just reads the VPC </hepta-color><hepta-color type="text" color="yellow">`subnet_ids`</hepta-color><hepta-color type="text" color="yellow"> to deploy, and an IAM Role ARN for CI/CD, without rethinking network and security.</hepta-color>
147	
148	<hepta-color type="text" color="yellow">These foundation pieces mark the shift from one-off experiments to platform thinking. We are not "starting machines" on AWS; we are building a software-defined platform with security, observability, and automation.</hepta-color>


## Note 12: Traffic entry diagram

- id: `2808956d-99d9-44c2-a3c5-c433a162449e`
- last_edited: 2026-11-07T02:25:24.483Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Traffic entry diagram" [2808956d-99d9-44c2-a3c5-c433a162449e] 7 lines
--- Databases ---
none
1	# Traffic entry diagram
2	
3	How a user request goes from DNS through the load balancer and cert layer to your app:
4	
5	![](heptabase://card/2808956d-99d9-44c2-a3c5-c433a162449e/file/f883eb0f-daf1-46c9-a30d-cbe2e27cd108)
6	
7	> *Full path into EKS: Route53 resolves the name, ALB terminates TLS with the ACM cert, then forwards the request.*


## Note 13: DNS and TLS Certificate Layer

- id: `39f10e66-3d06-4bc2-b848-179c652068ea`
- last_edited: 2026-11-04T16:04:40.438Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "DNS and TLS Certificate Layer" [39f10e66-3d06-4bc2-b848-179c652068ea] 68 lines
--- Databases ---
none
1	# DNS and TLS Certificate Layer
2	
3	After you create an AWS ALB you get something like `my-loadbalancer-1234567890.us-west-2.elb.amazonaws.com` — long, ugly, and it can change on rebuild. Users should not type that. Production needs a stable, branded entry such as `api.example.com`. That is what we build here: the **naming layer (Route53)** and the **certificate layer (ACM)**。
4	
5	## <hepta-color type="text" color="yellow">Route53: the platform naming layer</hepta-color>
6	
7	<hepta-color type="text" color="yellow">Cloud resources are highly dynamic. ALBs rebuild, IPs change. If upper services depend on raw endpoints, the system is brittle.</hepta-color>
8	
9	<hepta-color type="text" color="yellow">In our repo Route53 is more than DNS records. It is a </hepta-color><hepta-color type="text" color="yellow">**naming layer**</hepta-color><hepta-color type="text" color="yellow">.  Hosted zones map dynamic cloud resources onto stable URLs.</hepta-color>
10	
11	### External vs internal zones
12	
13	A production platform usually splits domains into two duties, each with its own hosted zone:
14	
15	1. **External zone (e.g. ****`*.example.com`****)**):
16	   Public users. Web UI, public APIs — names the global DNS must resolve.
17	
18	2. **Internal zone (e.g. ****`*.internal.example.net`****)**):
19	   Staff and admin tools (Grafana, DB consoles). Resolve only inside the VPC or via VPN. Safer (hides internals) and cleaner org-wise.
20	
21	These usually map to two **ACM wildcard certificates**: one for external traffic, one for internal.
22	
23	## ACM: automated digital trust
24	
25	Domain next, then HTTPS. Manual life was a nightmare: CSR, buy a cert, paste DNS validation, upload to AWS, renew every year by hand.
26	
27	Under IaC we use **AWS Certificate Manager (ACM)** plus Terraform to kill that workflow.
28	
29	### Automated DNS validation
30	
31	When you request a cert, AWS must prove you own the domain. ACM gives a CNAME (for example `_abc123.example.com` pointing at a validation target).
32	
33	In our Terraform this is a seamless handshake:
34	
35	1. **Terraform creates ****`aws_acm_certificate`**: request the cert, choose `DNS` validation.
36	
37	2. **Terraform reads validation data**: pull the ACM-generated CNAME from the cert resource.
38	
39	3. **Terraform writes Route53 records**: put that CNAME in the matching Hosted Zone.
40	
41	4. **ACM detects and issues**: once DNS is live, ACM issues the cert and auto-renews as long as that record stays.
42	
43	### Why a wildcard cert?
44	
45	We usually request `*.example.com` (wildcard). On EKS, services come and go. A wildcard gives new subdomains HTTPS immediately, without another Terraform cert request each time.
46	
47	## Zero-downtime updates:`create_before_destroy`
48	
49	A critical ops detail when handling certificates.
50	
51	Terraform's default is delete-then-create. If an ALB is using the cert, deleting it puts the ALB in a bad state and drops traffic.
52	
53	Add a `lifecycle` block on the ACM resource:
54	
55	```hcl
56	resource "aws_acm_certificate" "this" {
57	  domain_name       = "*.example.com"
58	  validation_method = "DNS"
59	
60	  lifecycle {
61	    create_before_destroy = true
62	  }
63	}
64	```
65	
66	This tells Terraform: when the cert must change (key rotation), **request and validate the new cert first, then delete the old one**. The load balancer always has a valid cert. Real zero downtime.
67	
68	<hepta-mention type="card" id="2808956d-99d9-44c2-a3c5-c433a162449e">Traffic entry diagram</hepta-mention>


## Note 14: EKS-dedicated CIDR → We allocate every pod via secondary CIDR in EKS

- id: `cda9e2f9-15c7-441f-b39b-805087553f53`
- last_edited: 2026-11-04T15:58:52.965Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "EKS-dedicated CIDR → We allocate every pod via secondary CIDR in EKS" [cda9e2f9-15c7-441f-b39b-805087553f53] 103 lines
--- Databases ---
none
1	# EKS-dedicated CIDR → We allocate every pod via secondary CIDR in EKS
2	
3	<hepta-color type="text" color="yellow">Imagine you are running a growing platform. At first you planned </hepta-color><hepta-color type="text" color="yellow">`10.0.0.0/16`</hepta-color><hepta-color type="text" color="yellow"> for the VPC and thought 65,536 IPs would be plenty. As microservices grow, something odd happens: you only have a few hundred EC2 instances, but available VPC IPs vanish by the thousands.</hepta-color>
4	
5	That is not magic. It is how **AWS VPC CNI** works: in EKS, <hepta-color type="text" color="yellow">by default every Pod takes a real VPC IP</hepta-color>. That gives excellent network performance, but it is also an IP black hole. When primary CIDR space runs out, how do you expand without rebuilding the whole network?
6	
7	## Why do Pods eat your IPs?
8	
9	In a classic Kubernetes model such as Flannel, Pods sit on an overlay isolated from the VPC, so thousands of Pods share a few node IPs. EKS's default <hepta-color type="text" color="yellow">EKS default </hepta-color><hepta-color type="text" color="yellow">**AWS VPC CNI**</hepta-color><hepta-color type="text" color="yellow"> gives Pods real in-VPC IPs for native performance.</hepta-color>
10	
11	That produces this IP-exhaustion math:
12	
13	1. **Pod count explosion**: 100 nodes with 40 Pods each consume 4,000 IPs immediately.
14	
15	2. <hepta-color type="text" color="yellow">**Warm pools**</hepta-color><hepta-color type="text" color="yellow">: to start Pods faster, VPC CNI pre-allocates several ENIs per node and pre-grabs a batch of IPs. Those IPs are unavailable even before Pods run.</hepta-color>
16	
17	3. <hepta-color type="text" color="yellow">**Infrastructure competition**</hepta-color><hepta-color type="text" color="yellow">: the primary CIDR also serves RDS (management IPs per subnet), ALBs, and other management EC2.</hepta-color>
18	
19	If the original plan was conservative, or the VPC was split into tiny subnets, you hit the wall fast. That is why we need a **secondary CIDR**。
20	
21	## The fix: RFC 6598 and 100.64.0.0/16
22	
23	When the main range is not enough, AWS lets you attach extra CIDRs to a VPC. For EKS, the industry practice is the address space defined by **RFC 6598**:`100.64.0.0/16`。
24	
25	Why this range?
26	
27	- **Carrier-grade NAT**: originally for telco large-scale NAT.
28	
29	- **Low collision**: most corporate networks (RFC 1918, such as `10.x.x.x` or `192.168.x.x`) do not use it, which cuts IP overlap on VPN or VPC peering.
30	
31	- **Dedicated use**: keep `10.x.x.x` for expensive databases and load balancers, and put tens of thousands of Pods in the `100.64.x.x` pool.
32	
33	### A dual-structure VPC
34	
35	The diagram shows this primary-vs-secondary layout:
36	
37	```
38	+-----------------------------------------------------------------------+
39	| VPC                                                                   |
40	|                                                                       |
41	| [Primary CIDR: 10.0.0.0/16]                                           |
42	| +-------------------------------------------------------------------+ |
43	| |  Management Subnets (Public/Private)                              | |
44	| |  - NAT Gateway, ALB, RDS, EC2 Instances (Node IPs)                | |
45	| +-------------------------------------------------------------------+ |
46	|                                                                       |
47	| [Secondary CIDR: 100.64.0.0/16] (EKS Pod-only space)                  |
48	| +-------------------------------------------------------------------+ |
49	| |  EKS Pod Subnets (Private)                                        | |
50	| |  - Used only by VPC CNI for Pod IPs                               | |
51	| |  - Pod A (100.64.10.5), Pod B (100.64.20.8)                       | |
52	| +-------------------------------------------------------------------+ |
53	|                                                                       |
54	+-----------------------------------------------------------------------+
55	```
56	
57	> <hepta-color type="text" color="yellow">*A VPC with both primary and secondary CIDRs. 10.0.x.x for foundation resources; 100.64.x.x reserved for Pods.*</hepta-color>
58	
59	## Terraform: attaching a CIDR
60	
61	You cannot change CIDR on the `aws_vpc` resource (that is primary and immutable). Attach dynamically with `aws_vpc_ipv4_cidr_block_association`.
62	
63	### 1. Create the association
64	
65	First tell AWS this VPC now has extra `100.64.0.0/16` territory.
66	
67	```hcl
68	resource "aws_vpc_ipv4_cidr_block_association" "eks_secondary" {
69	  vpc_id     = aws_vpc.main.id
70	  cidr_block = "100.64.0.0/16"
71	}
72	```
73	
74	### 2. Carve subnets in the new territory
75	
76	Then create subnets in the new range. Key point: **you must wait until the association finishes before creating subnets**, or AWS errors that the CIDR is not in the VPC.
77	
78	```hcl
79	resource "aws_subnet" "eks_pods" {
80	  for_each = var.availability_zones # assume you have an AZ map
81	
82	  vpc_id            = aws_vpc.main.id
83	  # split from the secondary CIDR
84	  cidr_block        = cidrsubnet(aws_vpc_ipv4_cidr_block_association.eks_secondary.cidr_block, 2, each.value.index)
85	  availability_zone = each.key
86	
87	  tags = {
88	    Name = "eks-pods-${each.key}"
89	    # important tags covered later
90	    "kubernetes.io/role/internal-elb" = "1"
91	  }
92	}
93	```
94	
95	## FAQ: does communication change?
96	
97	You may ask: if Pods live in `100.64.x.x` and RDS lives in `10.0.x.x`, can they still talk?
98	
99	Answer: **yes, with no extra gateway.**
100	
101	Inside a VPC, every CIDR — primary or secondary — has route-table `Destination` automatically pointing at `local`。 . Packets walk from living room to bedroom. No router or NAT.
102	
103	You get nearly unlimited IP headroom while keeping in-VPC communication fast and secure.


## Note 15: NAT Gateway: one-way exit for private networks

- id: `1a169772-bf53-4538-a178-2db7f8f4ab56`
- last_edited: 2026-11-04T10:44:14.368Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "NAT Gateway: one-way exit for private networks" [1a169772-bf53-4538-a178-2db7f8f4ab56] 21 lines
--- Databases ---
none
1	# NAT Gateway: one-way exit for private networks
2	
3	<hepta-color type="text" color="yellow">If the IGW is a two-way door, a </hepta-color><hepta-color type="text" color="yellow">**NAT Gateway**</hepta-color><hepta-color type="text" color="yellow"> is a one-way turnstile. Resources in a </hepta-color><hepta-color type="text" color="yellow">**Private Subnet**</hepta-color><hepta-color type="text" color="yellow"> (EKS nodes) can initiate outbound connections, but inbound from the internet is blocked.</hepta-color>
4	
5	### Why must a NAT Gateway sit in a public subnet?
6	
7	<hepta-color type="text" color="yellow">Beginners often find this confusing. A NAT Gateway serves private subnets, but it must live in a </hepta-color><hepta-color type="text" color="yellow">**Public Subnet**</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
8	
9	1. <hepta-color type="text" color="yellow">**Private subnet resources**</hepta-color><hepta-color type="text" color="yellow"> (EKS nodes) send traffic to the </hepta-color><hepta-color type="text" color="yellow">**NAT Gateway**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
10	
11	2. <hepta-color type="text" color="yellow">**NAT Gateway**</hepta-color><hepta-color type="text" color="yellow"> sits in a public subnet and has a route to the </hepta-color><hepta-color type="text" color="yellow">**Internet Gateway**</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
12	
13	3. <hepta-color type="text" color="yellow">**NAT Gateway**</hepta-color><hepta-color type="text" color="yellow"> forwards to the IGW, then the internet.</hepta-color>
14	
15	<hepta-color type="text" color="yellow">Put a NAT Gateway in a private subnet and it cannot find a way out. The path is:</hepta-color>
16	
17	- <hepta-color type="text" color="yellow">**Traffic path**</hepta-color><hepta-color type="text" color="yellow">：</hepta-color><hepta-color type="text" color="yellow">`Private subnet (EKS node) -> NAT Gateway (in a public subnet) -> Internet Gateway -> Internet`</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
18	
19	### Role of Elastic IP (EIP)
20	
21	A NAT Gateway is managed. External servers must recognize it and send replies, so it binds a **static, fixed** public IP, an **Elastic IP (EIP)**。  That matters in practice: vendors often require a whitelist IP. With an EIP, the EKS cluster's egress IP never changes.


## Note 16: Internet Gateway: the VPC front door

- id: `dc4a7e36-ff22-45c6-8019-a3098638d38d`
- last_edited: 2026-11-04T10:44:09.681Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Internet Gateway: the VPC front door" [dc4a7e36-ff22-45c6-8019-a3098638d38d] 15 lines
--- Databases ---
none
1	# Internet Gateway: the VPC front door
2	
3	<hepta-color type="text" color="yellow">**Internet Gateway (IGW)**</hepta-color><hepta-color type="text" color="yellow"> is the door between the VPC and the internet. Horizontally scaled, redundant, highly available. It carries traffic between your VPC and the public network.</hepta-color>
4	
5	<hepta-color type="text" color="yellow">In Terraform you create it with </hepta-color><hepta-color type="text" color="yellow">`aws_internet_gateway`</hepta-color><hepta-color type="text" color="yellow"> and attach it to the VPC. An IGW is not in a subnet. It attaches at VPC level.</hepta-color>
6	
7	### IGW traits and bidirectionality
8	
9	The IGW's core job is static NAT between subnet resources and public IPs. It is bidirectional:
10	
11	- <hepta-color type="text" color="yellow">**Outbound**</hepta-color><hepta-color type="text" color="yellow">: resources in the subnet can initiate internet access.</hepta-color>
12	
13	- <hepta-color type="text" color="yellow">**Inbound**</hepta-color><hepta-color type="text" color="yellow">: outsiders can reach resources with a public IP if the security group allows it.</hepta-color>
14	
15	When a subnet route table sends `0.0.0.0/0` to the IGW, that subnet is a **Public Subnet**。


## Note 17: Public vs Private: the Route Table

- id: `710f1037-2312-4b39-a626-c36bd86045af`
- last_edited: 2026-11-07T02:25:24.115Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Public vs Private: the Route Table" [710f1037-2312-4b39-a626-c36bd86045af] 90 lines
--- Databases ---
none
1	# Public vs Private: the Route Table
2	
3	Common myth: subnets are not born public or private. <hepta-color type="text" color="yellow">Public vs private is only the associated </hepta-color><hepta-color type="text" color="yellow">**Route Table**</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
4	
5	### Routing decision tree
6	
7	A packet leaving the subnet looks up the table.
8	
9	1. <hepta-color type="text" color="yellow">Dest inside the VPC → </hepta-color><hepta-color type="text" color="yellow">`local`</hepta-color><hepta-color type="text" color="yellow">, stay internal.</hepta-color>
10	
11	2. <hepta-color type="text" color="yellow">Dest is the Internet (</hepta-color><hepta-color type="text" color="yellow">`0.0.0.0/0`</hepta-color><hepta-color type="text" color="yellow">): the table decides everything</hepta-color>。
12	
13	![](heptabase://card/710f1037-2312-4b39-a626-c36bd86045af/file/d5bc4553-11fd-464e-9d58-67b0d79ae139)
14	
15	> *Leaving a subnet, dest IP + Route Table Target pick the path.*
16	
17	### 1. Public subnet
18	
19	- **Key config**: a rule sending `0.0.0.0/0` to **Internet Gateway (IGW)**。 
20	
21	- <hepta-color type="text" color="yellow">**Behavior**</hepta-color><hepta-color type="text" color="yellow">: resources with a Public IP (ALB, bastion) take Internet traffic and reply.</hepta-color>
22	
23	- <hepta-color type="text" color="yellow">**Use**</hepta-color><hepta-color type="text" color="yellow">: load balancers, public web servers.</hepta-color>
24	
25	### <hepta-color type="text" color="yellow">2. Private subnet</hepta-color>
26	
27	- <hepta-color type="text" color="yellow">**Key config**</hepta-color><hepta-color type="text" color="yellow">: </hepta-color><hepta-color type="text" color="yellow">`0.0.0.0/0`</hepta-color><hepta-color type="text" color="yellow"> points at </hepta-color><hepta-color type="text" color="yellow">**NAT Gateway**</hepta-color><hepta-color type="text" color="yellow">, or no Internet route at all.</hepta-color>
28	
29	- <hepta-color type="text" color="yellow">**Behavior**</hepta-color><hepta-color type="text" color="yellow">: no Public IP, no inbound from outside. They can only go out through NAT Gateway (updates, etc.).</hepta-color>
30	
31	- <hepta-color type="text" color="yellow">**Use**</hepta-color><hepta-color type="text" color="yellow">: EKS workers, databases, caches — the sensitive core.</hepta-color>
32	
33	## Batch subnets with for\_each
34	
35	Topic 1.5 `for_each`. In practice we do not write one `aws_subnet` per AZ. A map, Terraform iterates. Symmetric architecture.
36	
37	```hcl
38	locals {
39	  az_map = {
40	    "ap-northeast-1a" = 0
41	    "ap-northeast-1c" = 1
42	  }
43	}
44	
45	resource "aws_subnet" "private" {
46	  for_each = local.az_map
47	
48	  vpc_id            = aws_vpc.main.id
49	  availability_zone = each.key
50	  
51	  # index-driven CIDR: 10.0.0.0/24, 10.0.1.0/24...
52	  cidr_block = cidrsubnet(aws_vpc.main.cidr_block, 8, each.value)
53	
54	  tags = {
55	    Name = "private-subnet-${each.key}"
56	  }
57	}
58	```
59	
60	Data-driven network. Add an AZ: one Key-Value in `az_map`. Plan shows the new block.
61	
62	### Reserved addresses
63	
64	<hepta-color type="text" color="yellow">Every AWS subnet CIDR reserves </hepta-color><hepta-color type="text" color="yellow">**5 IPs**</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
65	
66	- <hepta-color type="text" color="yellow">`.0`</hepta-color><hepta-color type="text" color="yellow">: network address.</hepta-color>
67	
68	- <hepta-color type="text" color="yellow">`.1`</hepta-color><hepta-color type="text" color="yellow">: VPC router.</hepta-color>
69	
70	- <hepta-color type="text" color="yellow">`.2`</hepta-color><hepta-color type="text" color="yellow">: DNS.</hepta-color>
71	
72	- <hepta-color type="text" color="yellow">`.3`</hepta-color><hepta-color type="text" color="yellow">: AWS future use.</hepta-color>
73	
74	- <hepta-color type="text" color="yellow">`.255`</hepta-color><hepta-color type="text" color="yellow">: broadcast (VPC has no classic broadcast; still reserved).</hepta-color>
75	
76	A `/24` (256 theoretical) leaves **251** usable. Matter for large EKS.
77	
78	## Design philosophy
79	
80	VPC draws the line. Subnets spread risk and layer by security.
81	
82	- **VPC is the box**: IP ceiling for the project.
83	
84	- <hepta-color type="text" color="yellow">**Subnet is a room**</hepta-color><hepta-color type="text" color="yellow">: tied to one AZ.</hepta-color>
85	
86	- **Route Table is the manager**: can the room see the outside world?
87	
88	Terraform `cidrsubnet()` abstracts IP math. The foundation becomes predictable and easy to grow. First step from ClickOps to a platform.
89	
90


## Note 18: VPC and Subnet Implementation

- id: `9153a19a-b31b-4f97-9185-453fae30e6f3`
- last_edited: 2026-11-04T10:37:54.705Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "VPC and Subnet Implementation" [9153a19a-b31b-4f97-9185-453fae30e6f3] 77 lines
--- Databases ---
none
1	# VPC and Subnet Implementation
2	
3	You buy a plot in AWS wilderness: a VPC. Empty and sealed. You carve blocks (subnets): some face the public, some stay private. In Terraform this is math and layout, not clicks.
4	
5	## VPC: your private boundary
6	
7	A VPC is a logically isolated private network in AWS.  It is the foundation. No VPC, nowhere to put EC2, RDS, or EKS.
8	
9	Create one with `aws_vpc`. The key argument is `cidr_block` — how many IPs you own.
10	
11	```hcl
12	resource "aws_vpc" "main" {
13	  cidr_block           = "10.0.0.0/16"
14	  enable_dns_hostnames = true
15	  enable_dns_support   = true
16	
17	  tags = {
18	    Name = "main-vpc"
19	  }
20	}
21	```
22	
23	### Picking a CIDR
24	
25	<hepta-color type="text" color="yellow">Prod usually uses a private range like </hepta-color><hepta-color type="text" color="yellow">`10.0.0.0/16`</hepta-color><hepta-color type="text" color="yellow">. </hepta-color><hepta-color type="text" color="yellow">`/16`</hepta-color><hepta-color type="text" color="yellow"> means the first 16 bits are fixed; the remaining 16 (65,536 addresses) are yours.</hepta-color>
26	
27	**Why "logical" isolation?** Other AWS customers may use the same `10.0.0.0/16`. Virtualization keeps packets in your VPC. Same apartment building: everyone has a kitchen; you never see the neighbor cooking.
28	
29	## Subnet: physical split inside the region
30	
31	<hepta-color type="text" color="yellow">VPC is regional. A subnet lives in one Availability Zone. You cannot span two AZs with one subnet.</hepta-color>
32	
33	<hepta-color type="text" color="yellow">HA means at least two AZs (e.g. </hepta-color><hepta-color type="text" color="yellow">`ap-northeast-1a`</hepta-color><hepta-color type="text" color="yellow"> and </hepta-color><hepta-color type="text" color="yellow">`ap-northeast-1c`</hepta-color><hepta-color type="text" color="yellow">), each with its own subnet set.</hepta-color>
34	
35	### Terraform resource
36	
37	`aws_subnet` attaches to a VPC and takes its own CIDR (a subset of the VPC).
38	
39	```hcl
40	resource "aws_subnet" "public_1a" {
41	  vpc_id            = aws_vpc.main.id
42	  cidr_block        = "10.0.1.0/24"
43	  availability_zone = "ap-northeast-1a"
44	
45	  tags = {
46	    Name = "public-subnet-1a"
47	  }
48	}
49	```
50	
51	## The automation trick: cidrsubnet()
52	
53	Hand-slicing CIDR (`10.0.0.0/16` → `10.0.1.0/24`、`10.0.2.0/24`...) is dull and error-prone. Multi-env, multi-region hardcoding is an ops disaster.
54	
55	Terraform `cidrsubnet(prefix, newbits, netnum)` solves it.
56	
57	### Arguments
58	
59	1. **`prefix`**: source CIDR (usually the VPC).
60	
61	2. **`newbits`**: extra mask bits — subnet size. `/16` plus `8` bits → `/24`。
62	
63	3. **`netnum`**: which subnet, 0-indexed.
64	
65	Examples:
66	
67	- <hepta-color type="text" color="yellow">`cidrsubnet("10.0.0.0/16", 8, 0)`</hepta-color><hepta-color type="text" color="yellow"> -> </hepta-color><hepta-color type="text" color="yellow">`10.0.0.0/24`</hepta-color><hepta-color type="text" color="yellow"> (first /24)</hepta-color>
68	
69	- <hepta-color type="text" color="yellow">`cidrsubnet("10.0.0.0/16", 8, 1)`</hepta-color><hepta-color type="text" color="yellow"> -> </hepta-color><hepta-color type="text" color="yellow">`10.0.1.0/24`</hepta-color><hepta-color type="text" color="yellow"> (second /24)</hepta-color>
70	
71	- <hepta-color type="text" color="yellow">`cidrsubnet("10.0.0.0/16", 2, 3)`</hepta-color><hepta-color type="text" color="yellow"> -> </hepta-color><hepta-color type="text" color="yellow">`10.0.192.0/18`</hepta-color><hepta-color type="text" color="yellow"> (fourth /18)</hepta-color>)
72	
73	### Why it helps
74	
75	<hepta-color type="text" color="yellow">With </hepta-color><hepta-color type="text" color="yellow">`cidrsubnet()`</hepta-color><hepta-color type="text" color="yellow"> you generate blocks from a loop index (</hepta-color><hepta-color type="text" color="yellow">`count.index`</hepta-color><hepta-color type="text" color="yellow"> or a </hepta-color><hepta-color type="text" color="yellow">`for_each`</hepta-color><hepta-color type="text" color="yellow"> key). Grow 2 AZs to 3; the logic stays small.</hepta-color>
76	
77	<hepta-mention type="card" id="710f1037-2312-4b39-a626-c36bd86045af">Public vs Private: the Route Table</hepta-mention>


## Note 19: tfvars and Makefile

- id: `31fa1362-9290-4de5-b4ed-c6d9eb07ecf7`
- last_edited: 2026-11-04T10:20:46.132Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "tfvars and Makefile" [31fa1362-9290-4de5-b4ed-c6d9eb07ecf7] 105 lines
--- Databases ---
none
1	# tfvars and Makefile
2	
3	Last section covered **config/deployment separation**: blueprints (`.tf` files) and parameters (`.tfvars` files) live apart. That creates a practical problem: deploying a new EKS cluster to `prod` cannot mean typing a 200-character `terraform init` and `plan` every time.
4	
5	Imagine a typo in the S3 backend key at `init`, or forgetting the right `plan` on `-var-file`. You might create `int` resources in `prod` state — or worse, overwrite prod with test-sized parameters. To stop that human error we need a mechanical core.
6	
7	## `.tfvars` files: per-environment order forms
8	
9	If `deployments/` `.tf` files are the builder's blueprint, `config/` `.tfvars` files are the customer's order form.
10	
11	In our multi-environment layout, each environment × region pair has its own `.tfvars`. For example `int-us-west-2.tfvars` or `prod-ap-south-1.tfvars`。
12	
13	Compare int `int` with prod `prod`:
14	
15	```hcl
16	# int-us-west-2.tfvars (dev/integration)
17	env             = "int"
18	region          = "us-west-2"
19	cluster_version = "1.30"
20	
21	node_groups = {
22	  app_general = {
23	    instance_types   = ["t3.medium"]  # cheap, enough for tests
24	    desired_capacity = 1              # smallest size, save money
25	    max_capacity     = 2
26	  }
27	}
28	```
29	
30	Versus production:
31	
32	```hcl
33	# prod-ap-south-1.tfvars (live production)
34	env             = "prod"
35	region          = "ap-south-1"
36	cluster_version = "1.30"
37	
38	node_groups = {
39	  app_general = {
40	    instance_types   = ["c6a.xlarge"] # high-performance, compute-optimized
41	    desired_capacity = 3              # 3 AZs for HA
42	    max_capacity     = 10             # headroom for traffic spikes
43	  }
44	}
45	```
46	
47	The contrast is obvious: **same logic, wildly different size**. Developers change environment parameters in a simple variable file and never touch complex Terraform logic.
48	
49	## Makefile: the assembly heart
50	
51	With a blueprint (deployment) and an order form (config), we need an assembler. In this repo that is the **Makefile**.
52	
53	The Makefile is the mechanical heart of local and CI work. It takes three core variables: `ENVIRONMENT`、`REGION` and `COMPONENT` (eks or vpc). From those it derives every dynamic path:
54	
55	1. **DEPLOY\_PATH**: which deploy directory to enter, e.g. `deployments/service/eks/`。
56	
57	2. **Backend Config**: assemble the S3 bucket name, e.g. `project-prod-ap-south-1-tf-backend`。
58	
59	3. **State Key**: state path in S3, e.g. `prod/ap-south-1/service/eks/terraform.tfstate`。
60	
61	4. **Var File**: path to the right environment parameter file.
62	
63	So when you run `make tf_plan`, the Makefile assembles something like:
64	
65	```bash
66	# command the Makefile assembles
67	cd deployments/service/eks && \
68	terraform init \
69	  -backend-config="bucket=my-prod-backend" \
70	  -backend-config="key=prod/ap-south-1/service/eks/terraform.tfstate" && \
71	terraform plan \
72	  -var-file=../../../config/service/eks/prod-ap-south-1.tfvars
73	```
74	
75	## Why a Makefile? Predictability and foolproofing
76	
77	For beginners, typing `terraform plan` by hand feels fine. In production that manual feel is dangerous. The Makefile exists for **predictability**。
78	
79	One of the worst threats is **state overwrite**. If you are in `prod` and accidentally `init` the `int` backend, Terraform thinks it should "update" all `int` resources to look like `prod`. Test data gets force-overwritten with prod settings. Catastrophe.
80	
81	<hepta-color type="text" color="yellow">The Makefile (and later CI) cuts that risk:</hepta-color>
82	
83	- <hepta-color type="text" color="yellow">**Standardized paths**</hepta-color><hepta-color type="text" color="yellow">: backend paths are computed from variables, not typed by a human.</hepta-color>
84	
85	- <hepta-color type="text" color="yellow">**Parameter checks**</hepta-color><hepta-color type="text" color="yellow">: before Terraform runs, the Makefile or a helper script can check that the environment name is legal and the region exists.</hepta-color>
86	
87	- <hepta-color type="text" color="yellow">**Lower mental load**</hepta-color><hepta-color type="text" color="yellow">: operators do not memorize where each component's state lives. They only know which environment and which component.</hepta-color>
88	
89	## Flow diagram: GitHub to AWS
90	
91	The same automation is the foundation of GitHub Actions deploys:
92	
93	1. **GitHub Actions**: an engineer opens a PR and picks env, region, and component in GitHub.
94	
95	2. **Makefile**: the workflow calls the Makefile with those variables. The Makefile turns static text into Terraform commands with a real path and backend.
96	
97	3. **Terraform**: finally run `init`、`validate` and `plan/apply` to sync AWS.
98	
99	Parameterized pipelines mean whoever runs the deploy, wherever they are, the result is consistent and auditable. That is mature platform engineering: **standardized tools protect operators from mistakes.**
100	
101	## The automation foundation of multi-environment architecture
102	
103	The toolchain is complete: physically isolated state, logically split deployments, parameter-driven config, glued by a Makefile. Setup is a bit fussy at first, but expansion is cheap: a new region (for example `ap-southeast-1`) is a copied `.tfvars` and one variable at run time. No core code changes.
104	
105	After "how to deploy," the next question is: are we done? If someone bypasses Terraform and clicks in the AWS console, how do we know? Next: scheduled plans that detect configuration drift.


## Note 20: Config/Deployment Separation

- id: `d5e664ad-d5ad-4c5d-a243-973e2fc10047`
- last_edited: 2026-11-04T10:17:23.404Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Config/Deployment Separation" [d5e664ad-d5ad-4c5d-a243-973e2fc10047] 67 lines
--- Databases ---
none
1	# Config/Deployment Separation
2	
3	Imagine building three identical houses for three families, each with different paint, flooring, and furniture. Would you draw three full architectural plans? No. <hepta-color type="text" color="yellow">You keep one standard blueprint and a finish list per family. In Terraform, splitting structure from environment parameters is the mental model that keeps multi-environment work from becoming chaos.</hepta-color>
4	
5	## Why not copy-paste?
6	
7	You already split infrastructure into components such as VPC, EKS, or a database. That shrinks one state's blast radius. As the project grows, a new challenge appears: how do you create int, stg, and prod?
8	
9	The naive move — copy `vpc.tf` once per environment — quickly becomes ops hell:
10	
11	1. **Maintenance multiplies**: adding a VPC tag means editing three folders by hand.
12	
13	2. **Human error**: copy-paste misses an environment and architectures drift.
14	
15	3. **Hard to scale**: opening Singapore (ap-southeast-1) would mean a fourth and fifth copy of the code.
16	
17	<hepta-color type="text" color="yellow">Production projects use </hepta-color><hepta-color type="text" color="yellow">**config/deployment separation**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
18	
19	## The core pattern: blueprints and values
20	
21	Split the Terraform repo into two clear domains:
22	
23	### 1. `deployments/`: define the resource's shape
24	
25	This is pure logic. It says what a standard VPC contains (VPC object, three public subnets, three private, one NAT Gateway). <hepta-color type="text" color="yellow">The code is </hepta-color><hepta-color type="text" color="yellow">**environment-agnostic**</hepta-color><hepta-color type="text" color="yellow">. No hardcoded CIDRs or instance sizes — variables leave slots.</hepta-color>
26	
27	### 2. `config/`: define the resource's contents
28	
29	<hepta-color type="text" color="yellow">Per-environment variable files (</hepta-color><hepta-color type="text" color="yellow">`.tfvars`</hepta-color><hepta-color type="text" color="yellow">). The data that fills the blueprint. int uses </hepta-color><hepta-color type="text" color="yellow">`10.0.0.0/16`</hepta-color><hepta-color type="text" color="yellow">; prod uses </hepta-color><hepta-color type="text" color="yellow">`10.1.0.0/16`</hepta-color><hepta-color type="text" color="yellow">. No logic, only data.</hepta-color>
30	
31	<hepta-color type="text" color="yellow">That is </hepta-color><hepta-color type="text" color="yellow">**DRY (Don't Repeat Yourself)**</hepta-color><hepta-color type="text" color="yellow">: one copy of the logic; only parameters change.</hepta-color>
32	
33	## Directory layout
34	
35	Automation needs a strict naming convention to find the right blueprint and parameters.
36	
37	### Parameter naming
38	
39	<hepta-color type="text" color="yellow">Under </hepta-color><hepta-color type="text" color="yellow">`config/`</hepta-color><hepta-color type="text" color="yellow"> we organize module type > component > env-region:</hepta-color>
40	
41	```text
42	config/
43	+-- <module_type>/       <-- e.g. aws (foundation) or service (apps)
44	    +-- <component>/     <-- e.g. vpc, eks, rds
45	        +-- <env>-<region>.tfvars
46	```
47	
48	For example, int in us-west-2 for VPC lives at
49	
50	`terraform/config/aws/vpc/int-us-west-2.tfvars`
51	
52	### Logic naming
53	
54	Matching `deployments/` holds shared `.tf` files:
55	
56	```text
57	deployments/
58	+-- <module_type>/
59	    +-- <component>/
60	        +-- main.tf      <-- resource logic
61	        +-- variables.tf <-- slots
62	        +-- outputs.tf   <-- output API
63	```
64	
65	At a glance: change VPC architecture in `deployments/aws/vpc/`; add prod EKS nodes in `config/service/eks/prod-ap-south-1.tfvars`。
66	
67	<hepta-mention type="card" id="97d8af0c-b63e-4d08-843d-da9fc9161c95">Example: VPC</hepta-mention>


## Note 21: Split strategy: layer by change frequency and ownership

- id: `c7fe4dd6-4ab7-471c-8292-6a36f447bc4b`
- last_edited: 2026-11-04T10:11:10.836Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Split strategy: layer by change frequency and ownership" [c7fe4dd6-4ab7-471c-8292-6a36f447bc4b] 76 lines
--- Databases ---
none
1	# Split strategy: layer by change frequency and ownership
2	
3	In some enterprise repos, they will decide to split resources into two main classes:`aws/` and `service/`。
4	
5	### Foundation layer:`aws/`
6	
7	<hepta-color type="text" color="yellow">This layer is Foundation Stacks. It answers:</hepta-color><hepta-color type="text" color="yellow">**Which cloud primitives must exist for the platform to run?**</hepta-color><hepta-color type="text" color="yellow"> </hepta-color>
8	
9	- <hepta-color type="text" color="yellow">**Contains:**</hepta-color><hepta-color type="text" color="yellow"> </hepta-color><hepta-color type="text" color="yellow">**VPC**</hepta-color><hepta-color type="text" color="yellow">, </hepta-color><hepta-color type="text" color="yellow">**Subnets**</hepta-color><hepta-color type="text" color="yellow">, </hepta-color><hepta-color type="text" color="yellow">**Route53**</hepta-color><hepta-color type="text" color="yellow"> Hosted Zone, </hepta-color><hepta-color type="text" color="yellow">**ACM**</hepta-color><hepta-color type="text" color="yellow"> certificates, </hepta-color><hepta-color type="text" color="yellow">**global IAM Roles**</hepta-color><hepta-color type="text" color="yellow">, empty </hepta-color><hepta-color type="text" color="yellow">**Secrets Manager**</hepta-color><hepta-color type="text" color="yellow"> shells, and so on.</hepta-color>
10	
11	- <hepta-color type="text" color="yellow">**Traits:**</hepta-color><hepta-color type="text" color="yellow"> low frequency of change, strict permissions, the foundation for every upper service.</hepta-color>
12	
13	### Service layer:`service/`
14	
15	<hepta-color type="text" color="yellow">This layer is Workload-facing Stacks. It answers: </hepta-color><hepta-color type="text" color="yellow">**Which platform components or app infrastructure sit on that foundation?**</hepta-color><hepta-color type="text" color="yellow"> </hepta-color>
16	
17	- <hepta-color type="text" color="yellow">**Contains:**</hepta-color><hepta-color type="text" color="yellow"> EKS cluster, Node Groups, app-specific S3 buckets, RDS databases, and so on.</hepta-color>
18	
19	- <hepta-color type="text" color="yellow">**Traits:**</hepta-color><hepta-color type="text" color="yellow"> high frequency of change, tied to business logic, heavily dependent on resources from the </hepta-color><hepta-color type="text" color="yellow">`aws/`</hepta-color><hepta-color type="text" color="yellow"> layer</hepta-color>。
20	
21	This split gives each Module **its own State**. When you change an app service, you only run that Module's `plan/apply`, which touches tens of resources, not hundreds—fast and far safer.
22	
23	## Decision rule: should I split?
24	
25	<hepta-color type="text" color="yellow">For a new resource, use these two rules to decide if it should be its own Module (its own State):</hepta-color>
26	
27	1. **Is change frequency the same?** If A changes five times a week and B once a year, split them. Example: EKS Node Group (often) vs VPC (rarely).
28	
29	2. **What is the impact if it fails?** If A failing stops the company and B failing only breaks a small feature, split them. That isolates risk.
30	
31	### Data flow after the split
32	
33	You may ask: if VPC and EKS are split, how does EKS know which VPC Subnet to put nodes in?
34	
35	That is where last lesson's `terraform_remote_state` comes in.
36	
37	- **VPC Module** exposes `outputs.tf` `vpc_id` and `subnet_ids`。
38	
39	- **EKS Module** reads VPC outputs via `terraform_remote_state` and fills them into its own config.
40	
41	That builds a clear dependency chain: deploy `aws/vpc` first, take its outputs, then deploy `service/eks`。  Extra deploy-order complexity is worth the safety and speed.
42	
43	## Build a layered mindset
44	
45	Before implementation, keep this mental model. When we later create VPC, EKS, or databases, always ask: which layer? how often does it change? who should be allowed to edit it?
46	
47	| Comparison | Single giant State (anti-pattern) | Split layered architecture (best practice) |
48	| --- | --- | --- |
49	| **Speed** | Very slow; degrades linearly as resources grow | Very fast; only the affected Module runs |
50	| **Blast radius** | Global; one mistake can destroy everything | Local; impact stays inside one Module |
51	| **Collaboration** | Hard; State lock contention is common | Easy; different teams own different Modules |
52	| **Maintenance cost** | Low at first; later fear stops people from changing it | Slightly higher at first; later you change it with confidence |
53	
54	This architecture will carry us through the rest of the course and help you build a stable, scalable, production-grade cloud platform.
55	
56	## Physical layout of Modules
57	
58	In a real repository the directories look like this:
59	
60	```text
61	terraform/
62	└── deployments/
63	    ├── aws/          # Foundation: cloud primitives
64	    │   ├── vpc/      # Network foundation
65	    │   ├── route53/  # DNS hosting
66	    │   └── iam/      # Global identity
67	    └── service/      # Service layer: workloads
68	        ├── eks/      # K8s cluster
69	        └── database/ # Application database
70	```
71	
72	This structure draws a clear boundary for each component.  Every subdirectory under `deployments/` has its own `main.tf`、`variables.tf` and, most important, its own **Backend config** (its own State file).
73	
74	When developers change network settings, they know they are entering the high-risk area `aws/vpc`; when they only add an EKS node group, they stay in `service/eks`. Even a mistake in `eks` cannot touch Subnet settings in `vpc`, because `eks` State has no permission to change `vpc` resources.
75	
76	That physical isolation is a key step in professional IaC: automation is no longer a dangerous black box, but a controlled, predictable platform.


## Note 22: Remote Backend Bootstrap: Two-Step Migration Workflow

- id: `e1a3cd40-3278-4c0b-9ef3-ed7cef473318`
- last_edited: 2026-11-04T10:05:28.912Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Remote Backend Bootstrap: Two-Step Migration Workflow" [e1a3cd40-3278-4c0b-9ef3-ed7cef473318] 116 lines
--- Databases ---
none
1	# Remote Backend Bootstrap: Two-Step Migration Workflow
2	
3	> **Solving the Terraform Bootstrap Paradox**
4	>
5	> If Terraform needs an S3 bucket and DynamoDB table to store and lock State, but S3 and DynamoDB are also cloud infrastructure, how do you create them in the first place? Here is the standard two-step bootstrap lifecycle.
6	
7	## The Chicken-and-Egg Problem
8	
9	When starting a brand-new project, you cannot declare a Remote Backend in `backend.tf` right away because the target S3 bucket and DynamoDB lock table do not exist yet in AWS. If you try to run `terraform init` with a non-existent bucket, Terraform fails immediately.
10	
11	```text
12	[Chicken-and-Egg Paradox]
13	Need Remote Backend to store State -> But S3 bucket doesn't exist yet
14	Need Terraform to create S3 bucket -> But where does Terraform store its State?
15	```
16	
17	The solution relies on Terraform's built-in capability: **starting with Local State and migrating to Remote Backend**.
18	
19	---
20	
21	## The Standard Two-Step Migration Workflow
22	
23	### Step 1: Create Bootstrap Resources with Local State
24	
25	Isolate the backend infrastructure into a dedicated bootstrap directory (e.g., `bootstrap/` or `00-backend/`).
26	
27	1. Write Terraform code defining the S3 bucket (versioning + encryption enabled) and the DynamoDB table (partition key `LockID`).
28	
29	2. **Do not create ****`backend.tf`**** yet.** Leave the backend undefined.
30	
31	3. Run:
32	
33	   ```bash
34	   terraform init
35	   terraform apply
36	   ```
37	
38	   Terraform uses the default **Local State** (`terraform.tfstate` on your machine) and creates the S3 bucket and DynamoDB table in AWS.
39	
40	### Step 2: Add `backend.tf` and Migrate State to S3
41	
42	Now that the remote resources exist, configure Terraform to use them:
43	
44	1. Add `backend.tf` in the same bootstrap directory pointing to the newly created bucket and table.
45	
46	2. Run:
47	
48	   ```bash
49	   terraform init -migrate-state
50	   ```
51	
52	3. Terraform detects the backend change and prompts:
53	   `Do you want to copy existing state to the new backend?`
54	   Type `yes`.
55	
56	Terraform uploads your local `terraform.tfstate` to S3. From this point forward, even the bootstrap infrastructure's State is safely managed remotely in S3 with DynamoDB locking.
57	
58	### Step 3: Consumer Stacks Connect Directly
59	
60	With the Remote Backend live, all downstream modules (such as `vpc`, `database`, `eks`) can include `backend.tf` from their very first line of code:
61	
62	```text
63	           +---------------------------+
64	           |    00-bootstrap/          |
65	           | (Creates S3 + DynamoDB)   |
66	           +-------------+-------------+
67	                         |
68	            [Shared Remote Backend Ready]
69	                         |
70	       +-----------------+-----------------+
71	       |                                   |
72	       v                                   v
73	+---------------+                   +---------------+
74	|  network/vpc  |                   |  compute/eks  |
75	| (key: vpc...) |                   | (key: eks...) |
76	+---------------+                   +---------------+
77	```
78	
79	Each module uses a unique `key` path inside the same S3 bucket to isolate its blast radius.
80	
81	---
82	
83	## Prerequisite: How Terraform Authenticates to AWS
84	
85	Before running `terraform init` or `terraform apply`, Terraform must know which AWS account and region to talk to.
86	
87	### The AWS Credential Resolution Order
88	
89	Terraform (via the AWS Provider and S3 Backend) automatically searches credentials in this order:
90	
91	1. **Environment Variables** (Highest priority for CLI/scripts):
92	
93	   - `AWS_ACCESS_KEY_ID`
94	
95	   - `AWS_SECRET_ACCESS_KEY`
96	
97	   - `AWS_SESSION_TOKEN` (for temporary STS credentials)
98	
99	   - `AWS_REGION` or `AWS_DEFAULT_REGION`
100	
101	2. **Shared Credentials File** (`aws configure`):
102	
103	   - `~/.aws/credentials` and `~/.aws/config`
104	
105	   - Switch profiles using `export AWS_PROFILE=my-profile` or specifying `profile = "my-profile"` in HCL.
106	
107	3. **IAM Roles & OIDC (CI/CD & Kubernetes)**:
108	
109	   - In GitHub Actions: authenticated via OIDC (`aws-actions/configure-aws-credentials`).
110	
111	   - In EKS: authenticated via IRSA (IAM Roles for Service Accounts).
112	
113	   - On EC2: authenticated via Instance Profile (IMDS).
114	
115	If no valid AWS credentials exist in your environment when `backend "s3"` is configured, `terraform init` will fail with:
116	`Error: error configuring S3 Backend: no valid credential sources found.`


## Note 23: Remote Backend Setup

- id: `51c07e14-4235-47fc-8787-50b2659fb587`
- last_edited: 2026-11-07T02:25:22.263Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Remote Backend Setup" [51c07e14-4235-47fc-8787-50b2659fb587] 115 lines
--- Databases ---
none
1	# Remote Backend Setup
2	
3	> **The technical core of team coordination**
4	>
5	> With S3 plus DynamoDB, State is no longer a file on disk—it is a coordination service. That fixes conflicts and lays the ground for CI/CD.
6	
7	## Why local State is a collaboration landmine
8	
9	In the last chapter we learned that State is Terraform's memory. By default that memory lives in a `terraform.tfstate` file on your computer. Fine when you work alone. In a team, local State becomes a fatal single point of failure:
10	
11	1. **Version mismatch**: Developer A updates resources but forgets to pass the State file to developer B. B sees a stale view of infrastructure, which can recreate resources or delete the wrong ones.
12	
13	2. <hepta-color type="text" color="yellow">**Lost update**</hepta-color><hepta-color type="text" color="yellow">: If two people change the same resource at once, the last person to run </hepta-color><hepta-color type="text" color="yellow">`apply`</hepta-color><hepta-color type="text" color="yellow"> may overwrite the earlier work with no warning.</hepta-color>
14	
15	3. <hepta-color type="text" color="yellow">**Security risk**</hepta-color><hepta-color type="text" color="yellow">: State files hold sensitive data—database passwords, IAM role ARNs, internal IPs. Leaving that on a laptop is unsafe.</hepta-color>
16	
17	To get real IaC value, State must move from a personal file to a shared coordination mechanism.
18	
19	## What is a Backend?
20	
21	In Terraform, a **Backend** (backend) is a pluggable storage engine. It decides how Terraform loads, stores, and locks State. Think of it as a cloud drive: when you run `terraform plan` or `apply`, Terraform downloads the latest State from that backend and writes updates back when done.
22	
23	In AWS production, the most stable, recommended Backend combo is **S3 + DynamoDB**：
24	
25	- <hepta-color type="text" color="yellow">**S3 (Simple Storage Service)**</hepta-color><hepta-color type="text" color="yellow">: persistent storage for the State file.</hepta-color>
26	
27	- <hepta-color type="text" color="yellow">**DynamoDB**</hepta-color><hepta-color type="text" color="yellow">: locking, so only one person can change State at a time.</hepta-color>
28	
29	![](heptabase://card/51c07e14-4235-47fc-8787-50b2659fb587/file/7f4cf23d-7e51-4dec-aad1-0af0a51fca1f)
30	
31	> <hepta-color type="text" color="yellow">*This diagram shows how a Remote Backend coordinates a team: DynamoDB is a traffic light, S3 is the vault that holds the source of truth.*</hepta-color>
32	
33	## Best practices for S3 as Remote State storage
34	
35	Dropping State on S3 is not enough. Use S3's native features so this memory is both safe and recoverable:
36	
37	### <hepta-color type="text" color="yellow">1. Enable versioning</hepta-color>
38	
39	<hepta-color type="text" color="yellow">State files can corrupt. If a failed run makes State unreadable, S3 versioning lets you roll back to a healthy version. That is the last line of defense in an IaC incident.</hepta-color>
40	
41	### 2. Enforce encryption
42	
43	Because State holds secrets, the backend config must turn on `encrypt = true`. That encrypts State at rest in S3 (AES-256) so it cannot be stolen from storage.
44	
45	### 3. Bucket naming convention
46	
47	To manage many environments, use a consistent name format:
48	`<project>-<env>-<region>-tf-backend`
49	For example:`myproject-prod-us-west-2-tf-backend`. At a glance you know which environment and region that bucket holds.
50	
51	## <hepta-color type="text" color="yellow">DynamoDB State locking</hepta-color>
52	
53	<hepta-color type="text" color="yellow">Without a lock, two people running </hepta-color><hepta-color type="text" color="yellow">`apply`</hepta-color><hepta-color type="text" color="yellow"> at once causes a race; the State written to S3 can end up fragmented.</hepta-color>
54	
55	<hepta-color type="text" color="yellow">DynamoDB gives a simple, strong lock. When developer A runs a command, Terraform writes a record with </hepta-color><hepta-color type="text" color="yellow">`LockID`</hepta-color><hepta-color type="text" color="yellow"> into the named DynamoDB table.</hepta-color><hepta-color type="text" color="yellow"> When developer B tries next, Terraform checks the table, sees the lock, errors, and refuses to continue.</hepta-color>
56	
57	## Implementation: backend.tf
58	
59	To enable a Remote Backend in Terraform, create a `backend.tf` file. Note that the `terraform { backend "s3" { ... } }` block cannot use variables, because the Backend must initialize before variables load.
60	
61	```hcl
62	# deployments/aws/vpc/backend.tf
63	
64	terraform {
65	  backend "s3" {
66	    # S3 bucket name that stores State
67	    bucket         = "myproject-prod-us-west-2-tf-backend"
68	    
69	    # Path (key) of the State file inside the bucket
70	    # Suggested structure: <env>/<region>/<module_type>/<component>/terraform.tfstate
71	    key            = "prod/us-west-2/aws/vpc/terraform.tfstate"
72	    
73	    # AWS Region of the bucket
74	    region         = "us-west-2"
75	    
76	    # DynamoDB table used for locking (Partition Key must be LockID)
77	    dynamodb_table = "terraform-locks"
78	    
79	    # Enable S3 server-side encryption
80	    encrypt        = true
81	  }
82	}
83	```
84	
85	
86	### Key fields
87	
88	- **`bucket`**: your central warehouse. In a large project, each AWS account or environment usually has its own Backend bucket.
89	
90	- **`key`**: this is the State's **identity**。 A good path isolates components. For example, VPC and EKS State should live apart:
91	
92	  - `prod/us-west-2/aws/vpc/terraform.tfstate`
93	
94	  - `prod/us-west-2/service/eks/terraform.tfstate`
95	
96	- **`dynamodb_table`**: this table must exist first. Terraform can manage it, but we usually create this infrastructure-for-infrastructure by hand or with a separate bootstrap script.
97	
98	## Hierarchical State key naming
99	
100	In practice, infrastructure is split into stacks—network, database, app. We do not want one giant State for everything; that makes `plan` slow and too risky.
101	
102	So isolating via `key` is essential. As in this course repo's pattern:
103	`Stack identity = environment + region + module type + component name`
104	
105	That mapping gives each component its own memory space and lock scope, and greatly reduces the blast radius of changing A while accidentally hitting B.
106	
107	## Safety reminders
108	
109	State is in the cloud now, but do not forget:
110	
111	1. **`.gitignore`**** still matters**: even with a Remote Backend, Terraform still creates a local `.terraform/` directory to cache data. Add that path to `.gitignore` so local details do not leak into Git.
112	
113	2. **Access control**: lock down the S3 Backend bucket. Only the CI/CD service account and a few senior operators should have read/write.
114	
115	3. <hepta-color type="text" color="yellow">Always run </hepta-color><hepta-color type="text" color="yellow">`terraform init`</hepta-color><hepta-color type="text" color="yellow">: when you add or change </hepta-color><hepta-color type="text" color="yellow">`backend.tf`</hepta-color><hepta-color type="text" color="yellow">, you must run </hepta-color><hepta-color type="text" color="yellow">`terraform init`</hepta-color><hepta-color type="text" color="yellow">. Terraform detects the Backend change and asks whether to migrate existing State to the new remote location.</hepta-color>


## Note 24: The Essence of State

- id: `77308711-222f-418c-b8c5-29dd25ae4b50`
- last_edited: 2026-11-04T09:56:07.676Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "The Essence of State" [77308711-222f-418c-b8c5-29dd25ae4b50] 93 lines
--- Databases ---
none
1	# The Essence of State
2	
3	Imagine directing a construction robot. You give it a blueprint (code); it builds a wall. Next day you change the blueprint: tear down the wall, put in a window. How does the robot know which wall it built yesterday? And if a passerby paints the wall red while it sleeps, how does it notice?
4	
5	In Terraform, the answer is **State**. It is Terraform's memory, and the basis of every automated decision.
6	
7	## Terraform's brain: why State exists
8	
9	Last lesson we wrote HCL for resources, variables, and providers. You might think: we already defined resources in `.tf` files, so Terraform can just read them and check the cloud?
10	
11	Reality is harder. Terraform State is the core mechanism: it records which real resources this code manages. Without that memory, Terraform loses track of what it owns.
12	
13	Example: <hepta-color type="text" color="yellow">you define an AWS VPC named </hepta-color><hepta-color type="text" color="yellow">`main`</hepta-color><hepta-color type="text" color="yellow">. On first </hepta-color><hepta-color type="text" color="yellow">`terraform apply`</hepta-color><hepta-color type="text" color="yellow">, AWS returns a unique ID (e.g. </hepta-color><hepta-color type="text" color="yellow">`vpc-0123456789abcdef0`</hepta-color><hepta-color type="text" color="yellow">). Terraform must bind code </hepta-color><hepta-color type="text" color="yellow">`aws_vpc.main`</hepta-color><hepta-color type="text" color="yellow"> to cloud </hepta-color><hepta-color type="text" color="yellow">`vpc-0123456789abcdef0`</hepta-color><hepta-color type="text" color="yellow"> and store that in State. </hepta-color>
14	
15	Without that mapping, the next `apply` sees code saying "I want a VPC" but does not know yesterday's VPC is that one, so it may try to create another and collide.  State is not just a cache; it is the only bridge between intent and reality.
16	
17	## Three-way comparison: Code, State, and Reality
18	
19	To understand Terraform, learn the three-way comparison. Three parties:
20	
21	1. **Code (.tf)**: your desired state for infrastructure.
22	
23	2. **State file (.tfstate)**: Terraform's memory of managed state.
24	
25	3. **Real resources (AWS)**: the cloud's actual state.
26	
27	Differences among the three decide what `terraform plan` prints:
28	
29	### 1. Code vs State: what did you change?
30	
31	The usual path. You change a parameter in a `.tf` file (e.g. EC2 `instance_type` from `t3.micro` to `t3.small`). Terraform sees code ≠ State: intent changed, so it plans an update on next `apply`.
32	
33	### 2. State vs real resources: did someone change it by hand?
34	
35	The safety line. On `terraform plan`, <hepta-color type="text" color="yellow">Terraform first queries the AWS API and refreshes State. If cloud attributes differ from what State recorded, that is </hepta-color><hepta-color type="text" color="yellow">**Configuration Drift**</hepta-color><hepta-color type="text" color="yellow">。 </hepta-color>
36	
37	## Configuration Drift: when reality leaves the blueprint
38	
39	Take a VPC. Code sets CIDR `10.0.0.0/16`. A colleague, in a hurry, changes that CIDR in the AWS Console to `10.1.0.0/16` (or, more often, edits a Security Group).
40	
41	Next `terraform plan` does this:
42	
43	1. **Refresh**: Terraform calls the AWS API and sees the real CIDR is `10.1.0.0/16`。
44	
45	2. **Update State**: Terraform writes that discovery into the State file.
46	
47	3. **Compare and plan**: State now shows `10.1.0.0/16`, but code still says `10.0.0.0/16`。
48	
49	4. **Plan output**：`plan` shows an `~` (Update): it will change CIDR back to `10.0.0.0/16` to match the code.
50	
51	<hepta-color type="text" color="yellow">That is IaC's power: </hepta-color><hepta-color type="text" color="yellow">**code is the only source of truth**</hepta-color><hepta-color type="text" color="yellow">. Manual changes are detected and corrected on the next deploy. In production, regular </hepta-color><hepta-color type="text" color="yellow">`terraform plan`</hepta-color><hepta-color type="text" color="yellow"> is a health check: you see if anyone touched infra off-band.</hepta-color>
52	
53	## Disaster script: if we lose State
54	
55	If we accidentally delete the `terraform.tfstate` file, what happens?
56	
57	Terraform has amnesia. The VPC, subnets, and EKS cluster still run, but Terraform no longer remembers it built them.
58	
59	On the next `terraform apply`:
60	
61	1. Terraform scans `.tf` files and sees you want a VPC.
62	
63	2. It checks empty State: no record that it ever created that VPC.
64	
65	3. It tries to send AWS a create-VPC request.
66	
67	4. AWS errors on name/attribute conflicts, or you duplicate resources and double the bill.
68	
69	<hepta-color type="text" color="yellow">Then you must </hepta-color><hepta-color type="text" color="yellow">`terraform import`</hepta-color><hepta-color type="text" color="yellow"> resources one by one back into State. Painful and error-prone. Hence: </hepta-color><hepta-color type="text" color="yellow">**protect State as carefully as code — more carefully**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
70	
71	## State's dark side: security and secrets
72	
73	<hepta-color type="text" color="brown"><hepta-color type="background" color="yellow">State files hold a lot of sensitive data.</hepta-color></hepta-color>
74	
75	<hepta-color type="text" color="brown"><hepta-color type="background" color="yellow">Even if </hepta-color></hepta-color><hepta-color type="text" color="brown"><hepta-color type="background" color="yellow">`.tf`</hepta-color></hepta-color><hepta-color type="text" color="brown"><hepta-color type="background" color="yellow"> files use variables or Secret Manager, the State file (plain JSON) often stores those values in </hepta-color></hepta-color><hepta-color type="text" color="brown"><hepta-color type="background" color="yellow">**plaintext**</hepta-color></hepta-color><hepta-color type="text" color="brown"><hepta-color type="background" color="yellow">.</hepta-color></hepta-color>
76	
77	<hepta-color type="text" color="brown"><hepta-color type="background" color="yellow">Including:</hepta-color></hepta-color>
78	
79	- <hepta-color type="text" color="brown"><hepta-color type="background" color="yellow">Database bootstrap admin passwords</hepta-color></hepta-color>
80	
81	- <hepta-color type="text" color="brown"><hepta-color type="background" color="yellow">Env vars passed at create time (may include API keys)</hepta-color></hepta-color>
82	
83	- <hepta-color type="text" color="brown"><hepta-color type="background" color="yellow">IAM Role ARNs</hepta-color></hepta-color>
84	
85	- <hepta-color type="text" color="brown"><hepta-color type="background" color="yellow">Internal IPs and architecture details</hepta-color></hepta-color>
86	
87	Two hard bans:
88	
89	1. **Never commit ****`.tfstate`**** files to Git**. Exclude them in `.gitignore`.
90	
91	2. **Never casually share State files**。
92	
93	<hepta-color type="text" color="yellow">State is both critical and dangerous. How do we manage it safely? That is the next part:</hepta-color><hepta-color type="text" color="yellow">**Remote Backend**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>


## Note 25: The plan/apply Workflow

- id: `951e0748-effb-4cf1-9d74-ad1041546762`
- last_edited: 2026-11-04T06:30:34.136Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "The plan/apply Workflow" [951e0748-effb-4cf1-9d74-ad1041546762] 25 lines
--- Databases ---
none
1	# The plan/apply Workflow
2	
3	Imagine you operate a production database serving tens of thousands of users. Today you need to change instance type. Would you click Modify in the console and hope, or would you rather see a report first: will this cause downtime, which related resources are hit, and what the exact new params are?
4	
5	Terraform's `plan/apply` workflow exists to remove that uncertainty. It turns infra change from a one-shot command into a controlled, predictable, reviewable process.
6	
7	## Terraform's standard deploy cadence
8	
9	From first line of code to resources in the cloud, Terraform has a standard cadence. It is not just running commands; it is defensive design so desired state and the real world stay aligned.
10	
11	### 1. <hepta-color type="text" color="yellow">terraform init: set the stage</hepta-color>
12	
13	<hepta-color type="text" color="yellow">The first command in any Terraform directory. It initializes the environment. When you run </hepta-color><hepta-color type="text" color="yellow">`init`</hepta-color><hepta-color type="text" color="yellow">, Terraform scans your code, sees which providers you use (for example </hepta-color><hepta-color type="text" color="yellow">`hashicorp/aws`</hepta-color><hepta-color type="text" color="yellow">), and downloads the matching binaries from the official registry. It also initializes </hepta-color><hepta-color type="text" color="yellow">**Backend**</hepta-color><hepta-color type="text" color="yellow"> so there is a place for State (memory). </hepta-color>Without init, Terraform is a carpenter with no toolbox and cannot talk to the cloud.
14	
15	### <hepta-color type="text" color="yellow">2. terraform validate: static check</hepta-color>
16	
17	Before sending a plan to the cloud, confirm the code itself has no basic errors.`validate` checks HCL syntax, complete variable definitions, and resource references. It is a pure local check with no network. Like a compiler's first filter for typos and structural issues.
18	
19	### <hepta-color type="text" color="yellow">3. terraform plan: preview the future</hepta-color>
20	
21	The most critical step. `plan` computes the diff between current reality and the code target, and produces a change list. We will unpack this mental model next.
22	
23	### <hepta-color type="text" color="yellow">4. terraform apply: land the change</hepta-color>
24	
25	After you review `plan` and confirm, then execute. Locally, `apply` shows the `plan` output again and asks `yes` for approval. That human gate exists to prevent accidents.


## Note 26: Lifecycle: fine-tuning infra behavior

- id: `ea9b7724-fe02-4fac-98a1-a3632f43c853`
- last_edited: 2026-11-04T06:21:15.342Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Lifecycle: fine-tuning infra behavior" [ea9b7724-fe02-4fac-98a1-a3632f43c853] 103 lines
--- Databases ---
none
1	# Lifecycle: fine-tuning infra behavior
2	
3	Declarative Terraform is powerful and sometimes too mechanical. Default: an immutable attribute change deletes then recreates; remove a resource block and the next run deletes the cloud object.
4	
5	<hepta-color type="text" color="yellow">For real-world cases that do not fit the standard flow, Terraform offers the </hepta-color><hepta-color type="text" color="yellow">`lifecycle`</hepta-color><hepta-color type="text" color="yellow"> block. Think of it as a </hepta-color><hepta-color type="text" color="yellow">**fine manual knob**</hepta-color><hepta-color type="text" color="yellow"> beside automation, for edge cases the default path handles poorly. </hepta-color>These settings give infra more stability and more predictable behavior.
6	
7	### <hepta-color type="text" color="yellow">1. create\_before\_destroy: zero-downtime replace</hepta-color>
8	
9	Most-used option. When an update is a Replace—usually an AWS API that cannot update in place—Terraform's default order is Destroy then Create.
10	
11	For HA resources that means a gap. Set `create_before_destroy = true` and Terraform reverses: create the new resource, wait until it is ready, then delete the old one.
12	
13	**In practice: ACM cert rotation**
14	
15	<hepta-color type="text" color="yellow">For AWS ACM certs or ALBs, create-before-destroy keeps traffic up. Delete a cert still attached to an ALB and HTTPS dies immediately. You briefly pay for two certs during the swap. That cost is tiny next to an outage.</hepta-color>
16	
17	### <hepta-color type="text" color="yellow">2. prevent\_destroy: a hard safety lock</hepta-color>
18	
19	In IaC, delete one line and run `apply` and a prod database can vanish. Git has history; lost cloud data often does not.
20	
21	<hepta-color type="text" color="yellow">`prevent_destroy = true`</hepta-color><hepta-color type="text" color="yellow"> is a </hepta-color><hepta-color type="text" color="yellow">**physical lock**</hepta-color><hepta-color type="text" color="yellow"> on the resource. Once on, if you (or CI/CD) plan a destroy, Terraform errors at </hepta-color><hepta-color type="text" color="yellow">`plan`</hepta-color><hepta-color type="text" color="yellow"> and refuses to run. </hepta-color>
22	
23	<hepta-color type="text" color="yellow">This lock matters for:</hepta-color>
24	
25	- <hepta-color type="text" color="yellow">**Production databases (RDS, DynamoDB)**</hepta-color><hepta-color type="text" color="yellow">: stop a code typo from wiping data.</hepta-color>
26	
27	- <hepta-color type="text" color="yellow">**S3 buckets with real assets**</hepta-color><hepta-color type="text" color="yellow">: protect log or backup buckets from accidental delete.</hepta-color>
28	
29	- <hepta-color type="text" color="yellow">**Network foundation (VPC)**</hepta-color><hepta-color type="text" color="yellow">: delete a VPC and everything inside goes with it. Huge blast radius.</hepta-color>
30	
31	### 3. ignore\_changes: configuration drift
32	
33	Some attributes are not fully owned by Terraform. That is **Configuration Drift** (configuration drift): live cloud state does not match the code.
34	
35	Common cases:
36	
37	- **Secrets Manager**: in later security design, Terraform may only create a Secret shell (dummy value). The real secret (DB password) is filled by an admin or a rotator.
38	
39	- **Auto Scaling Group**: an ASG `desired_capacity` scales with traffic. If code says 3 and autoscaling hit 10, Terraform would force it back to 3—a prod incident.
40	
41	With `ignore_changes` you list attributes Terraform must ignore. Partial management: Terraform owns life cycle and base config; dynamic fields stay with other automation or people.
42	
43	### 4. replace\_triggered\_by: coupled rebuilds
44	
45	A more advanced knob. When resource A changes, force a rebuild of resource B.
46	
47	Useful when config is baked in. EC2 instances (B) depend on an init script in S3 (A). AWS will not reboot EC2 when the S3 object changes. `replace_triggered_by` tells Terraform: if the script changes, replace the old EC2 so they always run the latest.
48	
49	---
50	
51	## depends\_on: explicit resource order
52	
53	Terraform keeps a large dependency graph. Most of the time it draws itself.
54	
55	### Implicit dependency
56	
57	<hepta-color type="text" color="yellow">If one resource's HCL references another's output, Terraform creates the edge. </hepta-color><hepta-color type="text" color="yellow"> For example:</hepta-color>
58	
59	```hcl
60	resource "aws_vpc" "main" { ... }
61	
62	resource "aws_subnet" "private" {
63	  vpc_id = aws_vpc.main.id       # this references the VPC ID
64	}
65	```
66	
67	<hepta-color type="text" color="yellow">Terraform knows: build the VPC, get its ID, then build the Subnet. That is an </hepta-color><hepta-color type="text" color="yellow">**implicit dependency**</hepta-color><hepta-color type="text" color="yellow">, and it covers about 95% of ordering.</hepta-color>
68	
69	### Explicit dependency when implicit fails
70	
71	<hepta-color type="text" color="yellow">Sometimes HCL has no reference, but AWS still requires order. Implicit deps fail; you add </hepta-color><hepta-color type="text" color="yellow">`depends_on`</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
72	
73	#### Case: EKS secondary CIDR vs subnet race
74	
75	On production EKS, Pods eat many IPs, so we often add a secondary CIDR to the VPC (e.g. `100.64.0.0/16`）。
76	
77	Consider this:
78	
79	1. We define an `aws_vpc_ipv4_cidr_block_association` that attaches the new CIDR to the existing VPC.
80	
81	2. Then we create `100.64.x.x` subnets inside that `aws_subnet`。
82	
83	```hcl
84	resource "aws_vpc_ipv4_cidr_block_association" "eks_cidr" {
85	  vpc_id     = aws_vpc.main.id
86	  cidr_block = "100.64.0.0/16"
87	}
88	
89	resource "aws_subnet" "eks" {
90	  vpc_id     = aws_vpc.main.id
91	  cidr_block = "100.64.0.0/17"
92	
93	  # Key: add an explicit dependency
94	  depends_on = [aws_vpc_ipv4_cidr_block_association.eks_cidr]
95	}
96	```
97	
98	**Why ****`depends_on`****？**
99	Look at the `aws_subnet.eks` resource: it only references `aws_vpc.main.id`. Terraform thinks: VPC exists, so subnets can start.
100	
101	Inside AWS, if the `eks_cidr` association is not done, creating a subnet in that range fails: "CIDR block is not within the range of VPC". A race. With `depends_on` Terraform must wait for the association before creating subnets.
102	
103	---


## Note 27: Resource vs. Data ⇒ Look Up What Already Exists

- id: `56a72683-482e-41be-9bca-afd10a3460a3`
- last_edited: 2026-11-07T02:25:22.377Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Resource vs. Data ⇒ Look Up What Already Exists" [56a72683-482e-41be-9bca-afd10a3460a3] 148 lines
--- Databases ---
none
1	# <hepta-color type="text" color="yellow">**Resource vs. Data ⇒ Look Up What Already Exists**</hepta-color>
2	
3	Last section we used `resource`、`variable`、`locals` and `output` — the four building blocks — to describe a new vision: a VPC or Subnet from nothing.
4	
5	Reality is messier. You may drop a new EKS cluster into a VPC a teammate built last year, or you need the current AWS account ID for an IAM Policy. If Terraform can only create and never read, it is an architect with a blindfold.
6	
7	That is when **Data Source** and **Provider** show up: one is Terraform's eyes on reality; the other is the translator to the cloud.
8	
9	## Data Source: Terraform's eyes
10	
11	In Terraform a `resource` block creates and manages. A `data` block (Data Source) queries, read-only.
12	
13	In code terms, `resource` is like `new` an object into a database, and `data` is a `SELECT` statement. Data Sources fetch live cloud facts so you do not hardcode them.
14	
15	### Syntax
16	
17	A Data Source looks like a Resource; the keyword is `data`：
18	
19	```hcl
20	# syntax: data "<PROVIDER>_<TYPE>" "<LOCAL_NAME>" { ... }
21	
22	data "aws_vpc" "selected" {
23	  filter {
24	    name   = "tag:Name"
25	    values = ["production-vpc"]
26	  }
27	}
28	
29	# reference: data.<TYPE>.<LOCAL_NAME>.<ATTRIBUTE>
30	output "vpc_id" {
31	  value = data.aws_vpc.selected.id
32	}
33	```
34	
35	### Case 1: who am I? (aws\_caller\_identity)
36	
37	Almost every serious project uses this Data Source. Need the current Account ID, user ARN, or user ID? Do not type it. Ask AWS.
38	
39	```hcl
40	data "aws_caller_identity" "current" {}
41	
42	locals {
43	  account_id = data.aws_caller_identity.current.account_id
44	}
45	```
46	
47	<hepta-color type="text" color="yellow">Why it matters: S3 bucket policies and IAM roles often need </hepta-color><hepta-color type="text" color="yellow">`arn:aws:iam::${local.account_id}:root`</hepta-color><hepta-color type="text" color="yellow">. With a Data Source the same code moves across accounts (for example </hepta-color><hepta-color type="text" color="yellow">`int`</hepta-color><hepta-color type="text" color="yellow"> and </hepta-color><hepta-color type="text" color="yellow">`prod`</hepta-color><hepta-color type="text" color="yellow">) without editing the account ID.</hepta-color>
48	
49	### Case 2: find an existing network
50	
51	In large companies the network team already built the VPC. You get the account to deploy EKS. You should not (and often cannot) create another VPC. You find it.
52	
53	```hcl
54	data "aws_vpc" "shared_network" {
55	  tags = {
56	    Name = "Shared-VPC-Main"
57	  }
58	}
59	
60	resource "aws_eks_cluster" "main" {
61	  # use the looked-up VPC ID
62	  vpc_config {
63	    subnet_ids = data.aws_subnets.example.ids
64	  }
65	}
66	```
67	
68	**⚠️ Predict: what if a Data Source finds zero or more than one resource?**
69	<hepta-color type="text" color="yellow">A Data Source must return exactly one result. Zero matches or two VPCs with the same name and </hepta-color><hepta-color type="text" color="yellow">`terraform plan`</hepta-color><hepta-color type="text" color="yellow"> errors and stops. </hepta-color>That is a guardrail. Infra must not rest on a fuzzy guess.
70	
71	## Provider: the bridge to the cloud
72	
73	Terraform Core does not know AWS, GCP, or Azure. <hepta-color type="text" color="yellow">It parses HCL, builds the dependency graph, and manages State.</hepta-color>
74	
75	So how does AWS create an EC2 instance? **Provider**。
76	
77	<hepta-color type="text" color="yellow">A Provider is a separate binary plugin. It translates between Terraform Core and the cloud API.</hepta-color>
78	
79	![](heptabase://card/56a72683-482e-41be-9bca-afd10a3460a3/file/de424955-aa7e-47ce-a52c-1f0565bd7a40)
80	
81	> <hepta-color type="text" color="yellow">*Terraform Core talks to many cloud APIs through Providers — that is multi-cloud.*</hepta-color>
82	
83	### Why this architecture is smart
84	
85	1. **Decoupled updates**: AWS ships features weekly. The AWS Provider team can release often without waiting for Terraform Core.
86	
87	2. <hepta-color type="text" color="yellow">**Extensible**</hepta-color><hepta-color type="text" color="yellow">: anyone can write a Provider in Go — GitHub repos, Cloudflare, even smart bulbs.</hepta-color>
88	
89	## Version pinning: team consistency
90	
91	In `providers.tf` you usually see a `terraform` block listing Providers and versions.
92	
93	```hcl
94	terraform {
95	  required_providers {
96	    aws = {
97	      source  = "hashicorp/aws"
98	      version = "~> 5.0" 
99	    }
100	  }
101	}
102	```
103	
104	### Reading version operators `~>`
105	
106	<hepta-color type="text" color="yellow">Without a version, every </hepta-color><hepta-color type="text" color="yellow">`init`</hepta-color><hepta-color type="text" color="yellow"> downloads the latest Provider. Dangerous. New versions can break working code.</hepta-color>
107	
108	- `version = "5.10.0"`: pin exactly. Only this version runs.
109	
110	- `version = ">= 5.0"`: anything 5.0+. Still risky if 6.0 is a rewrite.
111	
112	- <hepta-color type="text" color="yellow">`version = "~> 5.0"`</hepta-color><hepta-color type="text" color="yellow">: the recommended form. Latest 5.x, never 6.0.</hepta-color>
113	
114	You get security fixes and minor features without major-version risk.
115	
116	## default\_tags: the lazy ops win
117	
118	Untagged AWS resources hurt billing and search. Writing `tags = { ManagedBy = "Terraform" }` on a hundred resources by hand is hell.
119	
120	The Provider has an elegant fix:`default_tags`。
121	
122	```hcl
123	provider "aws" {
124	  region = "us-west-2"
125	
126	  default_tags {
127	    tags = {
128	      Project   = "EKS-Platform"
129	      ManagedBy = "Terraform"
130	      Owner     = "Platform-Team"
131	    }
132	  }
133	}
134	```
135	
136	Once `default_tags` is set at the Provider, **every** tag-capable AWS resource (VPC, EC2, S3, security groups) gets those tags. Per-resource extra tags merge. That is governance consistency.
137	
138	## Summary: from vision to reality
139	
140	We now have Terraform's basic vocabulary:
141	
142	1. **Resource**: new resources we want to create.
143	
144	2. **Data Source**: existing facts we need to read.
145	
146	3. **Provider**: the cloud translator that runs the commands.
147	
148	You can write a full Terraform module: look up account info (Data Source), set region and default tags (Provider), declare network and compute (Resource).


## Note 28: gRPC vs REST

- id: `9cead154-fec3-4239-8d33-9ab666b201f8`
- last_edited: 2026-11-03T15:34:30.315Z
- whiteboards: go-backend

card "gRPC vs REST" [9cead154-fec3-4239-8d33-9ab666b201f8] 35 lines
--- Databases ---
none
1	# gRPC vs REST
2	
3	很多人對 gRPC 的第一印象是「二進位比較快」，但這只是表象。細節需要從傳輸、契約、串流三個維度來剖析。
4	
5	### 1. 傳輸協議：HTTP/2 vs. HTTP/1.1
6	
7	gRPC 強制使用 HTTP/2，而傳統的 REST API 絕大多數仍運行在 HTTP/1.1 上。
8	
9	- <hepta-color type="text" color="yellow">**二進位分幀（Binary Framing）**</hepta-color><hepta-color type="text" color="yellow">：REST (JSON) 是基於文字的（Text-based），解析 JSON 需要消耗較多 CPU；gRPC (Protobuf) 將訊息拆解為更小的二進位幀，解析效率極高。</hepta-color> [\[1\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/29146008-8955-4a62-8bd0-76fa8f43eaca?_agentLine=181#a0939902-33fb-496b-873b-4962d87daaab)
10	
11	- **多路複用（Multiplexing）**：<hepta-color type="text" color="yellow">**這是解決「隊頭阻塞」（Head-of-line Blocking）的關鍵。在 HTTP/1.1 中，如果你要發送多個請求，必須建立多個 TCP 連線或依序排隊；HTTP/2 允許在同一個 TCP 連線上同時交錯發送多個請求與回應，大幅減少了連線建立的開銷與延遲。**</hepta-color>
12	
13	- **標頭壓縮（HPACK）**：gRPC 的 HTTP/2 會壓縮重複的標頭，這對於頻繁呼叫、標頭欄位固定的微服務間通訊非常省頻寬。
14	
15	### 2. 契約（Contract）：強型別 vs. 弱檢查
16	
17	這是開發體驗（DX）與系統穩定性的分野。
18	
19	- **gRPC 基於 ****`.proto`**** 檔案定義契約**：這是一種 **Contract-first** 的設計。編譯器（protoc）會根據定義自動生成 Go 的 Client 與 Server Interface。這意味著如果欄位改了，編譯期就會報錯，而不是等到運行時才發現 JSON 欄位對不起來。 [\[2\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/29146008-8955-4a62-8bd0-76fa8f43eaca?_agentLine=182#a0939902-33fb-496b-873b-4962d87daaab)
20	
21	- **REST 依賴文件（OpenAPI/Swagger）**：這屬於 **Document-first**。雖然可以生成代碼，但它本質上是對「文字」的描述，缺乏像 gRPC 那樣嚴謹的型別約束。在金流系統中，一個金額欄位是 `string` 還是 `float64` 絕不能出錯，gRPC 的強型別能提供更好的保護。
22	
23	### <hepta-color type="text" color="yellow">3. 串流能力（Streaming）</hepta-color>
24	
25	<hepta-color type="text" color="yellow">gRPC 原生支援四種通訊模式：Unary（單向）、Client-side streaming、Server-side streaming 以及 </hepta-color><hepta-color type="text" color="yellow">**Bi-directional streaming（雙向串流）**</hepta-color><hepta-color type="text" color="yellow">。 </hepta-color><hepta-color type="text" color="yellow">[\[3\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/29146008-8955-4a62-8bd0-76fa8f43eaca?_agentLine=183#a0939902-33fb-496b-873b-4962d87daaab)</hepta-color><hepta-color type="text" color="yellow"> 在金流系統中，如果有一個需要長連線持續同步餘額或狀態的場景，gRPC 的雙向串流比 REST 頻繁 Poll 或使用複雜的 WebSocket 實作要簡潔得多。</hepta-color>
26	
27	## 場景選擇：什麼時候用什麼？
28	
29	### 內部服務 (Service-to-Service)：偏好 gRPC
30	
31	- **理由**：在 Circle 的內部架構（例如：支付引擎呼叫帳本服務），我們追求的是極致的**溝通效率**與**穩定性**。gRPC 的強型別契約能確保數十個微服務之間的接口一致性，自動生成的代碼能減少人為手寫 JSON Tag 或解析錯誤的風險。此外，在高併發的內部通訊中，HTTP/2 的多路複用能顯著降低連線池的壓力。 [\[4\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/29146008-8955-4a62-8bd0-76fa8f43eaca?_agentLine=185#a0939902-33fb-496b-873b-4962d87daaab) [\[5\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/29146008-8955-4a62-8bd0-76fa8f43eaca?_agentLine=189#a52148c9-bdfd-4c6f-8b9a-f4ac928853ea)
32	
33	### 外部介面 (Public API)：偏好 REST
34	
35	- **理由**：Circle 的願景是讓全球開發者都能接入 USDC 支付。這要求 API 必須具有**最低的進入門檻**。REST API 天生對瀏覽器友善，開發者只需要 `curl` 或 Postman 就能直接除錯，不需要先編譯 `.proto` 檔案。REST 的生態系廣泛，無論是 Web 端還是行動端，處理 JSON HTTP 請求都是最基本的能力。 [\[6\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/29146008-8955-4a62-8bd0-76fa8f43eaca?_agentLine=184-185#a0939902-33fb-496b-873b-4962d87daaab)


## Note 29: gRPC 負載平衡圖解：L4 vs L7

- id: `f64d8d4f-0131-4efd-a071-db5c904210fa`
- last_edited: 2026-11-03T15:34:10.623Z
- whiteboards: go-backend

card "gRPC 負載平衡圖解：L4 vs L7" [f64d8d4f-0131-4efd-a071-db5c904210fa] 99 lines
--- Databases ---
none
1	# gRPC 負載平衡圖解：L4 vs L7
2	
3	gRPC 在負載平衡上最著名的「長連線傾斜（Connection Imbalance）」問題
4	
5	## 核心矛盾：當「水管」遇到「多路複用」
6	
7	gRPC 之所以強大，很大程度歸功於 HTTP/2 的 **多路複用（Multiplexing）** 特性。 [\[1\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/29146008-8955-4a62-8bd0-76fa8f43eaca?_agentLine=181#a0939902-33fb-496b-873b-4962d87daaab) 傳統的 HTTP/1.1 在發送多個請求時，通常需要建立多條 TCP 連線，或者在同一條連線上排隊等待（Head-of-line blocking）。而 HTTP/2 允許在單一條 TCP 連線上同時發送成千上萬個並行的 **流（Streams）**，每個流代表一個獨立的請求與回應。 [\[2\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=7#623aebf8-e056-4535-89e3-52c153dd5bc4)
8	
9	從效能角度看，這非常優秀，因為它減少了建立 TCP 與 TLS 連線的開銷。但從負載平衡的角度看，這卻是一場災難。
10	
11	### L4 負載平衡：連線的「黏滯性」
12	
13	L4 代表 OSI 七層模型中的 **傳輸層（Transport Layer）**，常見的組件如 AWS NLB (Network Load Balancer) 或基本的 LVS。 L4 負載平衡器的運作邏輯非常單純：它只看 IP 地址和 Port。
14	
15	你可以把 TCP 連線想像成一根巨大的 **水管（Pipe）**，而內部的 HTTP/2 Streams 則是水管裡流動的一顆顆 **水滴（Drops/Requests）**。 [\[3\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=9#756c033a-5e6e-48aa-83fd-ec201bd524c4)
16	
17	<hepta-color type="text" color="yellow">當一個 Client 嘗試與 Service 通訊時，L4 負載平衡器會挑選一個後端 Pod（例如 Pod A），並在 Client 與 Pod A 之間建立一條 TCP 管道。一旦管道接通，負載平衡器就完成了它的任務，剩下的數據傳輸對它來說只是透明的二進位流。</hepta-color>
18	
19	<hepta-color type="text" color="yellow">**問題就在這裡：**</hepta-color><hepta-color type="text" color="yellow">
20	由於 gRPC 傾向於「</hepta-color><hepta-color type="text" color="yellow">**長連線**</hepta-color><hepta-color type="text" color="yellow">」，Client 一旦接通了 Pod A，後續的所有請求（水滴）都會沿著這根唯一的水管流向 Pod A。 </hepta-color><hepta-color type="text" color="yellow">[\[4\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=11#5b245ac1-9dfa-4d75-93fd-73d537c7e30b)</hepta-color><hepta-color type="text" color="yellow"> 就算你後來擴容了 Pod B 和 Pod C，既有的連線也不會主動斷開。</hepta-color>
21	
22	![這是一個網路架構示意圖，展示 L4 負載平衡在 gRPC 環境下的侷限性。採用教科書技術插圖風格，白色背景，線條清晰。左側有一個標籤為 "Client" 的方框，中間有一個代表 "L4 負載平衡器 (AWS NLB)" 的圓形圖示。右側垂直排列三個標籤分別為 "Pod A"、"Pod B"、"Pod C" 的方塊。Client 與負載平衡器之間連接一條粗實線。關鍵特徵是：從負載平衡器出發，只有一條粗實線連向 "Pod A"，而連向 "Pod B" 與 "Pod C" 的路徑以淡灰色虛線表示且無流量。在連向 "Pod A" 的粗實線內部，畫有多個小箭頭代表並行的 HTTP/2 Streams，全部流向同一個目標。圖中包含標註文字："TCP 連線 (長連線)"、"負載傾斜：Pod A 過熱"、"Pod B/C 閒置"。配色專業，避免過於鮮豔，強調流量分配的不對稱性。](heptabase://card/f64d8d4f-0131-4efd-a071-db5c904210fa/file/28a12641-f69e-4f98-be1c-03593e7ccb24)
23	
24	> *在 L4 負載平衡下，由於 LB 不具備解析 HTTP/2 幀的能力，導致所有多路複用的請求都固定在同一條 TCP 連線上，造成後端負載嚴重不均。* [\[4\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=11#5b245ac1-9dfa-4d75-93fd-73d537c7e30b)
25	
26	## L7 解決方案：拆解「水流」的藝術
27	
28	要解決這個問題，引導員必須變得「聰明」。他不能只看水管，他必須能看穿水管，觀察裡面的每一顆水滴。這就是 **L7 負載平衡（應用層，Application Layer）** 的職責。
29	
30	<hepta-color type="text" color="yellow">L7 負載平衡器（如 Envoy、NGINX 或 AWS ALB）能夠理解 HTTP/2 協議。它的運作方式與 L4 完全不同：</hepta-color>
31	
32	1. <hepta-color type="text" color="yellow">**終止與拆解**</hepta-color><hepta-color type="text" color="yellow">：L7 LB 會在自己這裡「終止」來自 Client 的 TCP 連線。它會解析 HTTP/2 內部的每一個二進位分幀（Binary Frames），將原本擠在一起的 Streams 拆解出來。 </hepta-color><hepta-color type="text" color="yellow">[\[5\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=19#139b784b-dfea-4360-b76a-1e7551ed6b0c)</hepta-color>
33	
34	2. <hepta-color type="text" color="yellow">**請求級別分發**</hepta-color><hepta-color type="text" color="yellow">：對於每一個拆解出來的 Request，LB 會根據負載平衡演算法（例如 Round Robin 或 Least Request），分別決定該送往哪個後端 Pod。</hepta-color>
35	
36	3. <hepta-color type="text" color="yellow">**重新封裝**</hepta-color><hepta-color type="text" color="yellow">：LB 會與後端 Pod 建立另一組連線池，將這些 Request 重新封裝並發送。</hepta-color>
37	
38	<hepta-color type="text" color="yellow">這樣一來，雖然 Client 只維持一條連線到 LB，但 LB 卻能把內部的請求均勻地灑在 Pod A、B、C 上。</hepta-color>
39	
40	![這是一個網路架構示意圖，展示 L7 負載平衡如何解決 gRPC 的分流問題。教科書技術插圖風格，白色背景。左側有一個標籤為 "Client" 的方框。中間是一個標註為 "L7 負載平衡器 (Envoy / ALB)" 的六角形圖示。右側垂直排列三個標籤為 "Pod A"、"Pod B"、"Pod C" 的方塊。Client 與 L7 負載平衡器之間維持一條粗實線。關鍵轉折在於：負載平衡器內部顯示出解析動作，從後端引出三條顏色不同的中型箭頭，分別均勻地指向 Pod A、Pod B 與 Pod C。每個箭頭代表獨立分發的 HTTP/2 Streams。圖中包含標註文字："HTTP/2 Streams 解析"、"Request-level 負載平衡"、"流量均衡分發"。整體色調柔和，對比明確。](heptabase://card/f64d8d4f-0131-4efd-a071-db5c904210fa/file/76f45216-96d9-48ba-8ba6-8c303ae67b49)
41	
42	> <hepta-color type="text" color="yellow">*L7 負載平衡器透過解析 HTTP/2 協定，將單一 TCP 連線內的請求拆解並分發到不同的後端服務，實現了真正的請求級負載平衡*</hepta-color>*。* [\[5\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=19#139b784b-dfea-4360-b76a-1e7551ed6b0c)
43	
44	## 實務架構建議：兩大主流路徑
45	
46	在 Kubernetes (K8s) 的環境中，處理 gRPC 負載平衡主要有兩種策略：
47	
48	### 1. 使用 Service Mesh (如 Istio/Envoy)
49	
50	這是目前大型企業（如 Circle 或 Trend Micro）最主流的做法。 [\[6\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=15#cce3f33c-bc9e-4f26-be53-90c2466dc906)
51	
52	- **機制**：在每個 Pod 旁邊部署一個 Sidecar Proxy（通常是 Envoy）。流量進入 Pod 前會先經過 Envoy。
53	
54	- **優點**：
55	
56	  - **透明性**：Go 程式碼完全不需要改動，它以為自己在跟一個普通的 Endpoint 通訊。
57	
58	  - **強大功能**：除了負載平衡，還能順便獲得重試、超時控制、熔斷與 OTel 打點。
59	
60	- **缺點**：
61	
62	  - **延遲稍微增加**：因為多經過了一層 Proxy。
63	
64	  - **運維複雜度**：需要管理整套 Service Mesh。
65	
66	### 2. Client-side Balancing (客戶端負載平衡)
67	
68	如果你對延遲極度敏感，不希望中間多過一層 Proxy，可以讓 Go Client 親自下場管理連線。
69	
70	- **機制**：Client 不再連接到一個單一的 VIP (Virtual IP)，而是向 DNS 查詢（通常是 K8s Headless Service）獲得所有後端 Pod 的 IP 列表。 [\[7\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=27#de2f949c-85ea-4523-a2a8-982df2e7ae91)
71	
72	- **作法**：Go 的 gRPC 客戶端庫內建了連線管理功能。它會對列表中的每個 IP 各自建立一條子通道（Subchannels），並在發送請求時使用內建的 `round_robin` 策略。 [\[8\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=29#3bc787b4-9583-4d0c-8643-b69a1ca5be2b)
73	
74	- **代碼範例**：
75	
76	  ```go
77	  // 使用 round_robin 策略建立連線
78	  conn, err := grpc.Dial(
79	      "dns:///my-service:8080",
80	      grpc.WithDefaultServiceConfig(`{"loadBalancingConfig": [{"round_robin":{}}]}`),
81	      grpc.WithTransportCredentials(insecure.NewCredentials()),
82	  )
83	  ```
84	
85	- **優點**：效能最高，沒有中間商。
86	
87	- **缺點**：Client 變得肥大，需要自己處理 Pod 擴容/縮容後的 IP 列表更新。
88	
89	## 為什麼我們仍需要多條 TCP 連線？
90	
91	你可能會問：「既然 L7 LB 已經幫我分流了，Client 與 LB 之間還需要多條 TCP 連線嗎？」
92	
93	答案是：**需要。** 即使有 L7 協助，我們通常仍會維持一個連線池（Connection Pool）或多條連線，原因如下：
94	
95	1. **解決 TCP 隊頭阻塞 (Head-of-line Blocking)**：這是 TCP 層的問題。如果某個 TCP 封包掉包了，整條連線都會卡住等待重傳。多條連線能讓一部分請求在其他連線上繼續跑。 [\[9\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=39#1b9d06f6-c027-4629-a3f8-ccb2ce5d0d87)
96	
97	2. **突破單核 CPU 限制**：單一 TCP 連線的處理（中斷、協議解析）通常會固定在單一 CPU 核心上。對於高吞吐量的服務，多連線能利用多核並行。 [\[10\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/083ad29e-530a-478a-a55c-965db759711b?_agentLine=41#be02ac7d-1fed-4ff1-8eae-ae15d729d9df)
98	
99	3. **安全性與熱點預防**：防止單一 LB 節點過載。


## Note 30: The Best Way to Use AI for Learning

- id: `a3964bef-29fb-4f14-ade7-da0ec74529f9`
- last_edited: 2026-11-03T15:18:14.059Z
- whiteboards: ♾️ Life

card "The Best Way to Use AI for Learning" [a3964bef-29fb-4f14-ade7-da0ec74529f9] 221 lines
--- Databases ---
none
1	# The Best Way to Use AI for Learning
2	
3	![](https://miro.medium.com/v2/resize:fit:10240/1*o2JqImz0-vEjxI9IHCYGXA.png)
4	
5	## 前言
6	
7	一直以來，我對自己的學習能力都抱著一定程度的自信，因為它讓我在人生的不同階段克服了許多挑戰，無論是求學（高中物奧全國前十、大一同時修碩博課程並獲書卷獎）還是工作（自學創業和打造軟體產品相關的知識，並將業務拓展至一百多個國家）。在這些歷程中，我幾乎不需改變學習方式，因為它始終有效且可靠。
8	
9	
10	然而，AI 的出現改變了一切。我意識到我原本的學習方法儘管有效，但在 AI 時代，它仍有巨大的進化空間。許多人使用 AI 來提升學習效率，例如將冗長的內容濃縮成重點，或將外文翻譯成母語，以便更快地完成原本就在做的事情。但我發現，這些看似細微的效率提升，當它們逐漸累積時，其實揭示了一個更深遠的改變：**許多過去因為太耗時而難以實踐的學習方式，如今突然變得可行了**。因此，<hepta-color type="background" color="orange">我認為最重要的問題並不是「如何更有效率地學習」，而是</hepta-color><hepta-color type="background" color="orange">**「如何讓自己有能力去學習更複雜、更抽象、也更具挑戰性的知識」**</hepta-color>。歸根究底，學習的價值並非在於累積盡可能多的知識，而是在於<hepta-color type="background" color="orange">培養能夠更深入思考重要問題的能力</hepta-color>。而往往正是這些最重要的問題，它們所需要的知識基礎也最為艱深。
11	
12	
13	以學習量子力學為例。閱讀最好的學術教科書可能需要 200 小時，對於只有 20 小時空閒時間的職場人士來說是非常不切實際的。在 AI 出現之前，你大概會把這 20 小時用在一些通俗的入門書籍、YouTube 影片或網路文章上。如今有了 AI，你有兩種選擇：
14	
15	1. 利用 AI 在 20 小時內消化更多相同類型的輕量內容 — — 比如再讀幾本通俗書籍、看更多影片和文章。
16	
17	2. <hepta-color type="background" color="orange">利用 AI 把這 20 小時直接用在學習該領域最好的學術教科書上。</hepta-color>
18	
19	
20	<hepta-color type="background" color="orange">我認為第一條路徑會帶來邊際效益遞減，而第二條則能在可控的時間內打開深入理解的大門。而且，在這個錯誤資訊橫飛的時代，我相信直接接觸一手資料比以往任何時候都更有價值。</hepta-color>
21	
22	在這篇文章中，我將一步步地分享我如何利用 AI 來自學教科書。在 AI 的幫助下，我明顯感受到在學習的深度和速度上都有顯著的提升，而我希望這篇文章能表達出我對此有多興奮。
23	
24	我選擇的書是 Christopher Bishop 的 *Pattern Recognition and Machine Learning* (PRML) — — 一本 710 頁的經典著作，在 Microsoft Research 的網站有提供[免費的 PDF 檔](https://www.microsoft.com/en-us/research/publication/pattern-recognition-machine-learning/)。我之所以選這本書，是因為它的目標讀者是進階大學生和博士班一年級學生，對於非相關背景的人來說通常有一定難度。我這次的示範會使用到前 32 頁的內容。雖然這是一本電腦科學的教科書，但我所介紹的方法適用於任何領域中內容厚重、需要長時間消化的知識文本。
25	
26	在附錄中，我也會分享如何利用 AI 來判斷哪些書值得投入時間，並提供一些合法且免費提供教科書的網站。不少人可能不知道的是，許多品質極高、甚至被視為領域中最好的教科書，其實都有在網路上發佈可開放取用的 PDF 檔。
27	
28	## 工具準備
29	
30	在解釋我如何用 AI 學習之前，以下是在我的學習過程中會用到的工具：
31	
32	- 我要學的教科書的 PDF 檔（如何取得可參見附錄）
33	
34	- 高品質的 PDF 解析器，具備 OCR 功能，能擷取文字、圖片、表格和公式
35	
36	- 先進的 AI 模型
37	
38	- 具備筆記功能的數位白板
39	
40	並不是一定要具備所有工具才能用 AI 輔助學習，但這些工具越完整，它們的效果就會互相加乘，讓你的學習能力提升得更明顯 — — 我稍後會透過實際的例子展示這一點。
41	
42	在這篇文章裡，我會用 [Heptabase](https://heptabase.com/) 作為示範工具。原因很簡單：這是我和團隊圍繞這種學習方式所打造給自己使用的產品，因此它已經內建了上述所有功能。你只需要準備好教材的 PDF 檔就行了。
43	
44	## 方法示範
45	
46	### 步驟一：解析 PDF
47	
48	當你有一份學習用的 PDF 檔時，<hepta-color type="background" color="orange">第一步是先使用 PDF 解析器把它</hepta-color><hepta-color type="background" color="orange">**轉換成通用格式**</hepta-color><hepta-color type="background" color="orange">。雖然現在很多 AI 工具都能讓你直接對 PDF 提問，但如果沒有這一步，多數情況下 AI 並不會把整份文件看完，而是只會挑選它覺得「相關」的部分 — </hepta-color>問題是，**AI 判斷相關性的方式（通常依靠 RAG 技術）其實並不可靠，可能會漏掉關鍵資訊。**
49	
50	透過轉換成通用格式，你就能自己決定 AI 在回答前必須讀過哪些內容，這會大幅提升回答的準確性和可靠性。比方說，你可以限定 AI 只根據某幾頁或特定段落回答，也可以請它翻譯或摘要指定的部分，而不用擔心遺漏重要資訊。
51	
52	在 Heptabase 裡，這件事很簡單：只要按下右上角的** Parse **按鈕，PDF 就會被轉換成通用格式，接著你在向 AI 提問時就能精準控制提供給 AI 的 PDF 範圍。
53	
54	<hepta-color type="background" color="orange">另外要注意，如果文件非常龐大（超過三萬字），記得開啟 </hepta-color><hepta-color type="background" color="orange">**MAX 模式**</hepta-color><hepta-color type="background" color="orange">，確保 AI 不會因為你給的內容過長而截斷資訊。最先進的 AI 模型在一段對話中能讀取的內容上限高達 40–100 萬個 tokens，相當於 30–75 萬字。《魔戒》三部曲大約有 48 萬字，也就是說你就算把整個三部曲全部放進對話裡，AI 依然能完整讀取並記住所有細節。</hepta-color>
55	
56	![](https://miro.medium.com/v2/resize:fit:2000/1*DEUVPS2m4bIYj9vt2ZgWdw.png)
57	
58	### 步驟二：製作學習素材
59	
60	在把 PDF 轉成通用格式後，下一步就是利用它來準備學習用的素材，並將這些學習素材放到白板上。你可能會好奇，為什麼不直接邊讀 PDF 邊和 AI 對話就好？當然，你確實可以這樣做。但對於篇幅較長（例如 710 頁的課本）、需要長時間閱讀和思考的知識內容，我認為預先準備好學習素材有非常大的價值（詳細的原因我會在步驟三和步驟四解釋）。最簡單的做法，就是把內容複製貼到卡片上，**然後依照書本的順序，在白板上從左到右排列。**一般來說，我會讓每張卡片代表一個章節；如果章節太長，也可以讓每張卡片代表一個小節。
61	
62	但如果這本書本身就不好讀 — — 比如用外文或文言文寫成、充滿專有名詞、或是假設你有某些你其實沒有的知識 — — 那麼一個更好的做法，就是請 AI 幫你把內容轉換成更有利於你吸收的學習素材，再放到白板上。
63	
64	轉換成什麼樣的學習素材最有效呢？你可以問自己二個問題：**第一，你在學習上的強項和弱項是什麼？第二，這份教材本身有哪些特性？**
65	
66	**例：**我讀英文的速度算一般，但中文閱讀非常快（因為我是台灣人）。不過在人名和專有名詞上，我讀英文的速度反而快很多。基於我的強項，我會請 AI 把正文翻譯成中文，但將所有人名和專有名詞保留為英文。
67	
68	> 請將所有內容一字不漏地翻譯成繁體中文，並且確保對中文讀者讀起來是通順的。人名要保持原文，專有名詞則要中英對照（以不影響可讀性為主）。請不要因為內容很長就分段翻譯。我要你一次一口氣全部翻譯完成，不用擔心輸出的內容太長，絕對不要遺漏任何一句話。
69	
70	
71	**例：**如果是歷史或人文類的書，我更喜歡先抓住整體架構、了解作者到底想表達什麼，接著才會去看細節。所以在這種情況下，我通常會請 AI 先幫我做一份符合我的需求的摘要，這份摘要就像我閱讀時的指南針，能幫助我在進入細節時不會迷路。以下是我可能會給 AI 下的 Prompt：
72	
73	> 請替還沒讀過本章的讀者寫一份章節摘要，整理出所有主要觀點、論證和主題，並說明它們是如何相互連結的。同時指出作者想解答的核心問題，並闡明整個章節的邏輯或敘事脈絡，讓讀者在進入正文前，就能先掌握這一章的思路。語氣要淺顯易懂，但也要保留作者觀點的深度。
74	
75	
76	有趣的是，摘要在學習數學時其實沒那麼有用。學一個新的數學主題更像是在學外語：真正的理解來自於親手去處理定義、結構和證明，而不是閱讀一個濃縮的摘要。對學習數學來說，更有幫助的是一份「關鍵定義清單」，每個定義附上一段直觀的解釋，再加上一些能激發好奇心的引導問題，讓我在進入抽象內容之前能先建立動機。
77	
78	
79	當你把這些學習素材生成好之後，就可以把它們放到白板上。在我這次的示範中，我不是直接複製原始內容，而是<hepta-color type="background" color="orange">請 AI 翻譯教科書的前 32 頁，並拆成 8 張卡片（綠色）</hepta-color>，每張卡片對應一個子章節或小節。
80	
81	我也請 AI 幫每個子章節做了摘要，整理出我最在意的關鍵定義、公式和結論。接著我把這些摘要訊息（白色）拖到白板上，放在翻譯好的卡片旁邊，讓我可以先讀摘要，再深入閱讀細節。
82	
83	![](https://miro.medium.com/v2/resize:fit:2000/1*YZhUvUnWJi0IjeYDP_SDTA.png)
84	
85	### 步驟三：從白板閱讀並與 AI 討論
86	
87	當我把翻譯好的卡片和摘要依照課本的順序從左到右排列在白板上時，光是這個佈局本身就已經非常有用。它讓我能夠一眼就看到課本這 32 頁的內容。
88	
89	為什麼這很有用呢？<hepta-color type="background" color="orange">傳統上，無論是讀實體書還是電子書，最令我沮喪的地方就是在跨頁時需要花費的時間與精力。假設我在第 105 頁讀到一個概念，意識到它和之前第 79 或 87 頁的內容有關聯，為了在腦中建立這些關聯，我得手動翻頁去找前面的內容，這過程雖然只需要幾秒，卻可能會打斷我當下的心流。更討厭的是我不能同時看到這些不同頁的內容！</hepta-color>
90	
91	但是當整整 32 頁的內容已經全部攤開在白板上時，我只需要移動視線，就能迅速找到先前做過的重點劃記。這只需要幾毫秒，意味著我可以很常這麼做而不會打斷心流。我很驚訝這樣微小的差別 — — 幾秒鐘對比幾毫秒 — — 竟然能對專注與閱讀心流產生如此巨大的影響。
92	
93	我想特別強調：這才是白板在學習過程中最能發揮價值的地方。許多人會把白板視為「視覺化」或「畫圖」的工具。他們會覺得自己要嘛是「視覺型思考者」（會用箭頭、心智圖、圖表），要嘛是「符號型思考者」（透過文章或程式碼表達想法）。但我認為這樣的二分法並沒有掌握到白板的本質。
94	
95	<hepta-color type="background" color="orange">**白板的本質是一張巨大、沒有邊界的書桌。**</hepta-color>無論你是否覺得自己是視覺型思考者，每個人都需要一張書桌。而你的書桌大小，決定了你同時能看到多少資訊，進而影響了你的思考能延伸到什麼程度。當然，聰明的人往往在腦中有一張很大的「虛擬書桌」 — — 也就是大腦的工作記憶，讓他們能在同一時間以更多資訊為基礎去思考。但當你面對資訊複雜度超過工作記憶容量的問題時，一張巨大的書桌、一個大螢幕，或是一個能容納龐大資訊的無限白板，就會成為最有用的工具之一。它可以把你從書本或電子閱讀器那個小小的矩形框裡解放出來。
96	
97	回到學習的過程：當我把書的內容整理成卡片放到白板上以後，我會開始閱讀這些卡片並劃記重點。這裡的步驟看起來並不特別，主要的好處來自於 AI 給的翻譯、摘要，以及白板所提供的資訊密度。但最關鍵的地方來了：**當我遇到困難、想要向 AI 提問時，它可以立刻掌握我當前所處的脈絡。**
98	
99	![](https://miro.medium.com/v2/resize:fit:2000/1*DQtoxaneVmaiHpfUWmpSjg.png)
100	
101	舉例來說，當我在讀「高斯分布」這張卡片時，裡面某個段落的解釋讓我感到困惑。這時：
102	
103	- 我可以直接從這張卡片右鍵開啟與 AI 的對話，並引用那段令我感到困惑的文字請 AI 解釋。AI 會以整張卡片的內容為脈絡來回答我的問題；
104	
105	- 我可以請 AI 現場舉一個淺顯易懂的例子來解釋這段文字給我聽、或是請 AI 設計一個假設性問題並用這段文字中所描述的知識來解決；
106	
107	- 我可以和 AI 分享我當前對這段文字的理解，並請它判斷是否正確，甚至請它出題測試我的理解程度；
108	
109	- 我甚至能讓 AI 讀取原本教科書的 PDF 檔，讓它在有完整脈絡之下告訴我作者寫這段文字的用意是什麼，這些細節是否在後面我還沒讀到的章節中變得重要。
110	
111	這感覺就像是隨身帶著一位已經讀過整本書的教授 — — 而且如果第一次得到的回答不理想，我還能換個問法重新得到更好的解釋。
112	
113	我最近用這個方式開始讀好幾本教科書，我的感想是，天啊，我真希望大學時代就有這種工具。以前一旦在讀課本時卡住，我常常得等到教授下一次的 office hour 才能解決。現在 AI 能讓我立即跨過讀書時碰到的每一個障礙。
114	
115	AI 和白板結合的另一個強大之處是：我可以把 AI 回覆的訊息直接拖到白板上，放在我從原文翻譯過來的卡片旁邊。這樣我之後隨時可以讓 AI 讀取整個白板，它不只能看到教科書的內容，也能看到我在這個白板上展開的每一個對話紀錄，進而更了解我的程度。
116	
117	Press enter or click to view image in full size
118	
119	![](https://miro.medium.com/v2/resize:fit:2000/1*ch_U_tNEfayndso0rNaJeQ.png)
120	
121	最後，白板上會同時呈現翻譯後的教科書內容（綠色卡片）、摘要、我的劃記和筆記，以及我和 AI 的完整問答記錄（白色訊息）。它成為一張無限的書桌，**讓我在學習過程中的完整脈絡和每一個思考的步驟清晰可見。**這意味著，如果我昨天讀了兩個小時，今天只要看一眼白板，就能立即回到當時的狀態，清楚知道該從哪裡繼續。
122	
123	### 步驟四：用自己的話做筆記
124	
125	假設我今天花了兩個小時閱讀（步驟三）。在閱讀結束、記憶還鮮明的時候，我一定會做筆記。我的筆記風格相當簡單：我會開一個空白筆記，嘗試把今天學到的內容用自己的話重寫，好像我要教給別人一樣。這個過程能幫助我發現並填補閱讀時沒有注意到的邏輯空隙。這和我一些教授朋友的做法類似 — — 當他們想學習一個新領域時，他們會直接開一門課來教這個領域。
126	
127	不過，所謂的「用自己的話重寫」其實不完全精確。實際上，我的做法是把這個空白筆記放在側邊欄，讓我可以同時看到它和白板。接著，我會把白板上我認為重要或有啟發性的內容 — — 無論來自教材卡片還是 AI 訊息 — — 複製貼上到筆記裡，再接著編輯、重塑這些我貼上的內容，用自己的敘事方式改寫。舉例來說，當我學習 *PRML* 時，我顯然沒有時間重打一遍所有的 LaTeX 公式。我通常都是直接將書的內容複製貼上（有時會做些修改），然後在這些公式下方加上自己的解釋。
128	
129	Press enter or click to view image in full size
130	
131	![](https://miro.medium.com/v2/resize:fit:2000/1*LMehd968Z5xKpdlvYurFqQ.png)
132	
133	這個編輯過程中一個關鍵環節是組織：我會將筆記依照主題整理成巢狀的條列結構。這能幫助我在腦中建立清晰的知識架構、找到概念之間的關聯。同時，它也迫使我審慎思考哪些內容是真正重要、值得保留的，哪些則可以刪掉。
134	
135	總而言之，我每天的閱讀基本上就是不斷重複步驟三 → 步驟四的過程：閱讀、發問、做筆記，然後繼續閱讀。經過幾天之後，當我這份筆記的內容累積得愈來愈多、可能包涵一整章或兩章的範圍時，我就會進入步驟五，開始視覺化地統整這些筆記。
136	
137	### 步驟五：視覺化與統整
138	
139	幾年前，我寫過一篇叫 [*The Best Way to Acquire Knowledge from Reading*](https://alanchan1209.medium.com/the-best-way-to-acquire-knowledge-from-readings-abf9357814d1)* *的文章，在裡面已經詳細解釋過步驟五的過程，所以我這裡不重複所有細節，只給一個簡短的版本：**當筆記變得非常長時，我會花時間把它拆解成更小的筆記，並視覺化它們之間的關係。**我發現這個過程對於加深理解非常有幫助。這也是畫箭頭和建立連結特別有用的階段。
140	
141	![](https://miro.medium.com/v2/resize:fit:2000/1*3qw3GlRsKQIzMUpXBv9wdQ.png)
142	
143	透過在白板上拆解和視覺化地重組筆記，我不僅能夠釐清單一本書的內容，也能更容易地把不同書籍與論文中的想法統整起來。當我不只是單純學習，而是在研究一個領域時，做這件事情特別有價值。
144	
145	## 結論
146	
147	現在我們可以回顧整個流程。如下圖所示，左邊大片的綠色區域代表我的學習素材與和 AI 的討論。每天完成當天的學習後，我都會把筆記記錄在同一張卡片上 — — 也就是中間的黃色卡片。當這張卡片累積到一定的內容量時，我會把它拆分成多張小卡片並將其視覺化，形成右邊的藍色區域。
148	
149	![](https://miro.medium.com/v2/resize:fit:9488/1*73nSMOb1S3QANhMAn4qr8A.png)
150	
151	PDF 解析器、白板與 AI 的整合，讓我在學習與思考進階教材時，能夠更好地維持心流狀態。而我相信這僅僅只是開始 — — 在這個學習過程中，我注意到其實有很多部分都可以透過開發 AI Agent 來自動化（特別是第二步驟「建立學習素材」）。隨著 AI 的進步與功能的演化，我們將愈來愈接近「AI 老師」的時代 — — 未來 AI 將不僅能理解最艱深的教科書，還能直接在白板上把知識教給你。這才是最令我感到興奮的地方！
152	
153	許多人說 AI 是炒作，也有人認為過度依賴 AI 等於「外包思考」，會讓人失去思考能力。但我的經驗完全相反。對我來說，AI 反而讓我能挑戰更艱深的教材，並投入更多時間去思考。它是我遇過最好的老師與思考夥伴，而我對「好老師」的標準一向非常嚴格。
154	
155	在學習這件事上，「外包」根本沒有意義。AI 幫你解出一百道數學題，不會讓你的數學更好。但如果你在這一百道題裡遇到瓶頸時請 AI 給你一些提示，反而能讓你保持動力並堅持下去。如何使用好像 AI 這樣的工具，最終還是取決於你自己。
156	
157	當我創立 Heptabase 時，我的夢想是打造一個最強大的學習與研究工具，讓能我學習任何我感興趣的主題，不論它有多困難。這也是為什麼我們的網站一直以來的標語都是：**「理解複雜的主題。」**
158	
159	表面上，Heptabase 可能看起來只是一個白板和筆記軟體。但在它的核心，每一項功能都是為了同一個目的所設計：幫助人們處理龐大、艱深的知識內容。過去幾年，我們為了這個願景打造了許多功能，但直到近期 AI 的突破，我才真正感覺到我們離實現這個夢想已經非常接近。能生活在這樣一個特別的時代，實在是一件幸運的事。
160	
161	最後，我想引用 Douglas Engelbart 在 *Augmenting Human Intellect (1962) *中的一段話，它非常精準地表達了我對 AI 如何改變學習的看法。簡單來說：許多高階的學習與解決問題的方法，只有在初階能力變得快速而高效之後，成本上才會變得可行。而多虧了 AI，這些初階能力如今已比歷史中任何時候都來得更快、更有效率。
162	
163	> *在某一項具體能力上的直接創新，可能會對整個能力階層產生深遠影響。這種變化可以向上傳遞：更高層的能力能夠利用這項被改變的能力，於是它們可以重新組織，充分發揮這項變化以及中間層能力的提升。這種變化也可以向下傳遞：高層的新能力會讓低層原本隱藏的潛在能力被激發並發揮作用。這些能力在過去可能無法發揮作用，但因為高層能力的改變，現在變得可用。*
164	>
165	> *\[…] 為了實現「積極提升人類智力效能」這個目標，關鍵在於我們要能夠貫穿整個能力階層，持續探索並推進這些「新可能性的鏈條」（這需要一種系統性的方法）。同時，我們也要意識到，新的能力往往來自於對既有與新興能力的重新組織與整合（這需要一種系統工程式的方法）。*
166	
167	## 附錄：如何找到最好的書籍與合法的 PDF
168	
169	在學習新的主題時，最容易被低估但卻至關重要的一步，就是<hepta-color type="background" color="orange">挑選好的教材</hepta-color>。若你接下來幾週要花 20 小時以上閱讀一本書，那麼花些時間研究該讀哪一本，將會是非常值得的投資。
170	
171	在搜尋書籍時 — — 無論是教科書或其他書籍 — — 我總是先從詢問 AI 開始。一個設計良好的提示（prompt）能帶來巨大差異。好消息是，你不需要是寫提示的專家；相反地，你可以請 AI 幫你設計提示。例如，我可能會先從這樣的問題開始：
172	
173	> 如果我想要請 AI 幫我找到學習機器學習的最佳教科書，我應該使用怎樣的提示才是最好的？
174	
175	經過幾輪來回修正後，我可能會得到如下的版本：
176	
177	> 你是一位機器學習領域的專家研究員與教師。我希望找到最適合用來學習機器學習的唯一最佳教科書。請先列出機器學習的主要學派（例如：統計學習、機率建模、深度學習、理論／計算學習理論等）。對於每個學派，請推薦一本最具權威性且在教學上最有效的教科書，並解釋為什麼它最能代表該學派。
178	>
179	> 接著，請比較這些學派在以下面向的差異：
180	>
181	> \- 它們在現代機器學習中的重要性，
182	>
183	> \- 在直覺與嚴謹數學解釋之間的平衡程度，
184	>
185	> \- 以及在建立深度專業能力上的長期價值。
186	>
187	> 最後，請分析其中的取捨，並建議我應該從哪個學派開始學習，因此應該先讀哪一本教科書，並清楚說明為什麼它最符合我的背景與學習風格。
188	
189	一旦確定了提示，就將它丟給最好的模型並開啟「思考模式」。最近我喜歡使用 GPT-5 Thinking，但最佳模型通常每隔幾個月就會變動。
190	
191	不要盲目接受 AI 的第一個答案。以下是一些交叉檢驗的方法：
192	
193	- **詢問多個模型**：看看哪些書籍反覆被提及。
194	
195	- **研究作者背景**：他們是否是該領域的領軍人物？是否任教於頂尖大學，或在產業中有重大貢獻？他們最重要的著作是什麼？
196	
197	- **檢查實際使用情況**：該書是否被頂尖大學用作教科書？
198	
199	- **查看社群回饋**：瀏覽 Goodreads、Reddit 或其他論壇 — — 讀者是否覺得這本書有價值且易讀？
200	
201	- **詢問專家或可信賴的朋友**，特別是對該科目有深厚功力的人。
202	
203	挑選書籍的一個實用準則是：選一本對你來說「剛好有點難以企及」的書。理想狀態下，它應該是那種若靠自己單打獨鬥會覺得太難，但在博士級導師指導下卻能勉強應付的書。
204	
205	- 如果你需要在閱讀每一句話時都依賴 AI，那這本書可能對你來說太難了。
206	
207	- 如果你只需要每隔幾段才求助 AI，那這本書可能比較適合你，因為它在過去對你來說太過困難，但如今你可以靠 AI 的協助來跨越理解時遇到的障礙。
208	
209	一旦選定了書籍，接下來可以檢查它是否有合法的開放存取 PDF。
210	
211	- [Project Gutenberg](https://www.gutenberg.org/) 提供超過 75,000 本文學、哲學、科學及其他領域的公共著作。
212	
213	- 對於計算機科學／數學領域，我蠻推薦 [FreeTechBooks](https://www.freetechbooks.com/about)。這個網站上的書籍全都由作者或出版社官方網站公開提供。
214	
215	- 一些經典著作有專門的網站。例如，[The Feynman Lectures on Physics](https://www.feynmanlectures.caltech.edu/) 以及 [David Hume](https://davidhume.org/) 的所有著作，都有專屬網站可免費瀏覽所有內容。
216	
217	- 也可以查看作者的個人網站。許多作者會在上面分享他們撰寫的教科書的免費檔案。
218	
219	如果沒有可用的 PDF，另一個選項是購買書籍掃描器。你可以先買下實體書，再將它掃描成 PDF 供自己學習使用，並嚴格限制在教育用途上 — — 務必要遵守所在國家的著作權法規。
220	
221	<hepta-bookmark tokenId="107" nodeId="c04c5808-278b-4f63-8041-ab62cc3739c8">[The Best Way to Use AI for Learning](https://medium.com/heptabase/the-best-way-to-use-ai-for-learning-762c3467bdf1)</hepta-bookmark>


## Note 31: 白板展開與即時提問

- id: `19cac42c-a716-4c06-969c-66d6547059e4`
- last_edited: 2026-11-03T15:15:12.040Z
- whiteboards: (not placed)

card "白板展開與即時提問" [19cac42c-a716-4c06-969c-66d6547059e4] 9 lines
--- Databases ---
none
1	# 白板展開與即時提問
2	
3	- 將教材卡片從左到右排列在白板上，能讓人一眼掌握數十頁的內容，在跨頁比對概念時無需翻頁即可無縫連接。[\[13\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=87-91#c301fdef-a007-4e91-b5fa-6cc98de9af5c,0efc1a6a-c2d4-45d8-8128-1450ac06eca4,82453554-947a-48b6-9c35-83f8ee945171)
4	
5	- 當閱讀卡片遇到疑惑時，可直接選取文字開啟與 AI 的對話，AI 會以整張卡片為脈絡提供深入且符合上下文的解答。[\[14\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=101-104#00a6eacf-d15e-421b-b6be-b6d55a7d132b,4e89869b-53ce-4a55-99e4-e06e47319cb0)
6	
7	- 讀者可以請 AI 舉出生活化例子、設計情境問題或出題測驗，甚至還能讀取原書 PDF 釐清作者編寫段落的深層用意。[\[15\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=105-110#d2f91e56-021c-4b87-bd80-22daf0523873,5fe5008f-a400-4bb0-875e-374abd0b1f47,b1a47e77-9e1b-4175-bdce-26b44c439c73)
8	
9	- 與 AI 討論後的訊息可直接拖曳回白板排列在卡片旁，不僅隨時保留思考軌跡，也讓 AI 後續讀取白板時更了解使用者的理解程度。[\[16\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=115-116#02fe9f8e-4931-43fe-8a8d-3c5b30ba2ae1)


## Note 32: 挑戰複雜知識能力

- id: `1b1b9971-cc15-49b7-885f-fee1eef63a55`
- last_edited: 2026-11-03T15:15:12.040Z
- whiteboards: (not placed)

card "挑戰複雜知識能力" [1b1b9971-cc15-49b7-885f-fee1eef63a55] 9 lines
--- Databases ---
none
1	# 挑戰複雜知識能力
2	
3	- AI 時代最重要的問題不是「如何更有效率地學習」，而是「如何讓自己有能力去學習更複雜、更抽象、也更具挑戰性的知識」。[\[1\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=10#77f00f0b-0119-4b59-87a7-c1101ba8ec22)
4	
5	- 學習的價值不在於累積盡可能多的知識，而是在於培養能夠更深入思考重要問題的能力。[\[1\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=10#77f00f0b-0119-4b59-87a7-c1101ba8ec22)
6	
7	- 面對有限的時間，用 AI 消化輕量內容會帶來邊際效益遞減，而直接投入該領域最好的學術教科書則能打開深入理解的大門。[\[2\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=13-20#867aa1ed-dac7-4041-a215-2c2b234b95c2,12885244-1e22-4afd-94c0-324a7f9f49b9,21a7498f-65ec-49fd-a510-3e3240cf8bf7,9e56492c-be5e-49bf-afad-62664be4de5b)
8	
9	- 學習時不應將思考外包給 AI，而是利用它在遇到瓶頸時提供提示，藉此保持動力並挑戰更高階的教材。[\[3\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=153-155#af017a84-694e-47b8-a03a-d99f1c3b5a17,f411a938-d4dc-4b29-b3bf-35b6f4327abd)


## Note 33: 挑選剛好難以企及的難度

- id: `4109ccf4-af19-4055-b47c-fb2efd1e1e3a`
- last_edited: 2026-11-03T15:15:12.040Z
- whiteboards: (not placed)

card "挑選剛好難以企及的難度" [4109ccf4-af19-4055-b47c-fb2efd1e1e3a] 9 lines
--- Databases ---
none
1	# 挑選剛好難以企及的難度
2	
3	- 挑選書籍時的一項實用準則是選擇對自己來說「剛好有點難以企及」的教材。[\[22\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=203#93ac3091-12d3-44eb-9a74-6d2708cccd38)
4	
5	- 理想的難度是若靠個人單打獨鬥會感到太困難，但在具備博士級導師指導下能勉強應付的深度。[\[22\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=203#93ac3091-12d3-44eb-9a74-6d2708cccd38)
6	
7	- 如果閱讀過程中的每一句話都必須依賴 AI 解釋，則代表該教材的難度可能已經超出自身能力過多。[\[23\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=205#d0ae68a3-09e4-4bce-b830-c5df4242a7b1)
8	
9	- 最適合的書籍是只需每隔幾段求助 AI 一次，藉由 AI 跨越過去難以獨自克服的理解障礙。[\[24\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=207#163b3b34-322f-491a-8240-5ee3624c4400)


## Note 34: 無限白板作為延伸桌面

- id: `82bdcf5b-9170-45b1-90d4-7439691e0a12`
- last_edited: 2026-11-03T15:15:12.040Z
- whiteboards: (not placed)

card "無限白板作為延伸桌面" [82bdcf5b-9170-45b1-90d4-7439691e0a12] 7 lines
--- Databases ---
none
1	# 無限白板作為延伸桌面
2	
3	- 白板的本質並非單純的視覺化繪圖工具，而是一張沒有邊界的巨大書桌，決定了同時能審視的資訊總量。[\[19\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=93-95#6e01460c-9601-4547-afc4-c6dd0d1b7d7c,d573fa81-5230-4865-a882-0ec547ea3808)
4	
5	- 當面對高度複雜且超過大腦工作記憶容量的知識時，無限白板能打破螢幕與書頁的框限，協助思考向外深度延伸。[\[20\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=95#d573fa81-5230-4865-a882-0ec547ea3808)
6	
7	- 白板完整並列了教材內容、摘要、個人註記與 AI 問答紀錄，使學習過程與思考步驟隨時清晰可見，隔日也能秒速進入心流繼續閱讀。[\[21\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=121#8cd99ca0-4001-4ace-813f-ef0c9ba0f0a4)


## Note 35: 解析 PDF 通用格式

- id: `bec869eb-3a42-4e5e-a747-93bb517e28ab`
- last_edited: 2026-11-03T15:15:12.040Z
- whiteboards: (not placed)

card "解析 PDF 通用格式" [bec869eb-3a42-4e5e-a747-93bb517e28ab] 7 lines
--- Databases ---
none
1	# 解析 PDF 通用格式
2	
3	- 使用學習用的 PDF 時，第一步是使用 PDF 解析器將其轉換成通用格式，避免直接提問時 AI 因 RAG 機制而漏掉關鍵資訊。[\[4\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=48#4454c0a3-f781-4d4d-90be-705569fbf667)
4	
5	- 轉換成通用格式後，使用者能夠自主決定並限定 AI 在回答前必須閱讀的頁面與段落範圍，顯著提升回答的準確度。[\[5\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=50-52#78fbb7f0-4c50-4a07-92ba-4b73ff055c07,2ed6f510-9c58-49ef-a38b-ac91d9f5e06d)
6	
7	- 面對超過三萬字的龐大文件，開啟 MAX 模式可確保模型在極高 token 上限下完整讀取且不截斷任何內容細節。[\[6\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=54#58ddda9e-b624-4b6e-9475-0c1912064b02)


## Note 36: 客製化學習素材

- id: `e8d82a36-1f57-44cc-9ef3-b2c070885636`
- last_edited: 2026-11-03T15:15:12.040Z
- whiteboards: (not placed)

card "客製化學習素材" [e8d82a36-1f57-44cc-9ef3-b2c070885636] 11 lines
--- Databases ---
none
1	# 客製化學習素材
2	
3	- 根據個人的學習強弱項與教材特性，預先請 AI 將內容轉換為易於吸收的學習素材再放上白板。[\[7\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=62-64#826d6597-213e-4b6f-89a5-d693e8ff962c,83223ce3-c988-4226-8b02-ef76ec51a8db)
4	
5	- 針對外語或難讀文本，可要求 AI 一字不漏地翻譯為母語，並保留人名與專有名詞以加速閱讀。[\[8\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=66-68#972e4a83-7036-4aee-88ab-f0fe01c5bdae,f0a7e53a-a997-4e16-906f-89b2406faace)
6	
7	- 人文與歷史書籍適合先由 AI 生成掌握核心論證的整體架構摘要，以作為閱讀細節時的指南針。[\[9\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=71-73#6a8b9a2d-3dfc-4d4f-9463-dc579d426a0c,800d19c1-eb63-489e-ba4a-4639140557e5)
8	
9	- 學習數學時摘要效果有限，更需要關鍵定義清單、直觀解釋與引導問題來建立理解動機。[\[10\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=76#5784234d-507a-424d-84a6-c65e71ff80aa)
10	
11	- 生成素材後依照章節順序在白板上由左至右展開卡片，能一眼掌握整體內容並維持閱讀心流。[\[11\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=60-60#250e231b-b4de-4c12-83d6-f8f50bd61696)[\[12\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a3964bef-29fb-4f14-ade7-da0ec74529f9?_agentLine=79-87#e97637fd-ce98-42d2-b3e6-1653c55de4c0,5fd4b031-80cb-4a63-97c1-15cd4960e0d7,c11c8432-0161-4943-809b-fac89d444d9a,59514dc6-9cea-475a-8fcf-25e1b3f049de,c301fdef-a007-4e91-b5fa-6cc98de9af5c)


## Note 37: A multi-environment mindset

- id: `a28e1012-2655-4829-8dfa-1d12822ec95c`
- last_edited: 2026-10-31T07:29:07.802Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "A multi-environment mindset" [a28e1012-2655-4829-8dfa-1d12822ec95c] 5 lines
--- Databases ---
none
1	# A multi-environment mindset
2	
3	You now split resources into components to isolate risk, and reuse one blueprint with many parameter sets. That is the production platform foundation.
4	
5	In practice we do not cd around or type long paths. Next we build an assembly line with **Makefile** that, from env and region, pairs the right blueprint with the right parameters for a safe, efficient deploy.


## Note 38: Example: VPC

- id: `97d8af0c-b63e-4d08-843d-da9fc9161c95`
- last_edited: 2026-10-31T07:28:57.592Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Example: VPC" [97d8af0c-b63e-4d08-843d-da9fc9161c95] 54 lines
--- Databases ---
none
1	# Example: VPC
2	
3	What this looks like in practice.
4	
5	### 1. In `deployments/aws/vpc/` create a shared blueprint
6	
7	This `main.tf` has no environment names. It only declares structure. Concrete numbers come from `var.*`：
8	
9	```hcl
10	# deployments/aws/vpc/main.tf
11	
12	resource "aws_vpc" "main" {
13	  cidr_block = var.vpc_cidr  # from a variable
14	  
15	  tags = {
16	    Name = "${var.env}-${var.region}-vpc" # assemble the name dynamically
17	    Environment = var.env
18	  }
19	}
20	
21	# other subnet, internet gateway, ... definitions
22	```
23	
24	### 2. In `config/aws/vpc/` fill in environment parameters
25	
26	A `.tfvars` file per environment:
27	
28	```hcl
29	# config/aws/vpc/int-us-west-2.tfvars
30	env      = "int"
31	region   = "us-west-2"
32	vpc_cidr = "10.10.0.0/16"
33	```
34	
35	```hcl
36	# config/aws/vpc/prod-ap-south-1.tfvars
37	env      = "prod"
38	region   = "ap-south-1"
39	vpc_cidr = "10.20.0.0/16" # production uses a completely different CIDR
40	```
41	
42	To create int, tell Terraform: use the logic in `deployments/aws/vpc/` with parameters from `config/aws/vpc/int-us-west-2.tfvars`.
43	
44	## Why this split helps
45	
46	Ops efficiency jumps:
47	
48	- **Fast expansion**: add `stg`? Add one `.tfvars` file. Zero new `.tf` code.
49	
50	- **Visible diffs**: compare int and prod by comparing two small parameter files, not thousands of lines of code.
51	
52	- **Safety isolation**: shared logic, independent parameter files, plus independent state from last section. Changing int cannot touch prod.
53	
54	<hepta-mention type="card" id="a28e1012-2655-4829-8dfa-1d12822ec95c">A multi-environment mindset</hepta-mention>


## Note 39: terraform_remote_state Cross-Module References

- id: `f323e647-e7b8-44e7-b146-72bb56cf681b`
- last_edited: 2026-10-31T07:21:22.875Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "terraform_remote_state Cross-Module References" [f323e647-e7b8-44e7-b146-72bb56cf681b] 39 lines
--- Databases ---
none
1	# terraform\_remote\_state Cross-Module References
2	
3	<hepta-color type="text" color="yellow">In the world of Terraform, we likewise split infrastructure into different stacks (e.g., VPC, EKS, database), each with its own independent state file. This "divide and conquer" strategy dramatically reduces the </hepta-color><hepta-color type="text" color="yellow">**blast radius**</hepta-color><hepta-color type="text" color="yellow">, but it also creates a new challenge: how does one stack "see" another stack's data? That is when </hepta-color><hepta-color type="text" color="yellow">`terraform_remote_state`</hepta-color><hepta-color type="text" color="yellow"> enters the picture.</hepta-color>
4	
5	## The Costs and Benefits of Splitting State
6	
7	Why don't we write all our resources into one giant `main.tf`?
8	
9	1. **Blast radius control**: if you put the VPC and the production database in the same state, and one day you only want to change a VPC tag, but a Terraform bug or a mistaken operation causes the database to be replaced, that would be catastrophic. After splitting, modifying the VPC's state never touches the database's resources.
10	
11	2. **Execution efficiency**: when the state contains hundreds of resources, every `terraform plan` must query the cloud provider for the current status of every resource, which becomes very slow. After splitting, a stack typically has only 20-30 resources, so plan finishes in seconds.
12	
13	3. <hepta-color type="text" color="yellow">**Separation of concerns**</hepta-color><hepta-color type="text" color="yellow">: the network team operates </hepta-color><hepta-color type="text" color="yellow">`aws/vpc`</hepta-color><hepta-color type="text" color="yellow">, and the platform team operates </hepta-color><hepta-color type="text" color="yellow">`service/eks`</hepta-color>, so their development workflows do not interfere with each other.
14	
15	However, the cost of splitting is **information isolation**. When an EKS cluster is created, it must know the VPC's exact `subnet_ids` to place worker nodes in the right locations. We cannot hardcode these IDs in the EKS code, because IDs change from one environment to another. We need a safe, automated way to read data across state boundaries.
16	
17	## <hepta-color type="text" color="yellow">terraform\_remote\_state: A Read-Only Bridge Across States</hepta-color>
18	
19	<hepta-color type="text" color="yellow">`terraform_remote_state`</hepta-color><hepta-color type="text" color="yellow"> is a special data source. Unlike ordinary data sources (e.g., </hepta-color><hepta-color type="text" color="yellow">`aws_ami`</hepta-color><hepta-color type="text" color="yellow">) that query cloud APIs, it directly </hepta-color><hepta-color type="text" color="yellow">**reads the state file another stack keeps on S3**</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
20	
21	<hepta-color type="text" color="yellow">Its operating logic is quite intuitive:</hepta-color>
22	
23	- <hepta-color type="text" color="yellow">It is </hepta-color><hepta-color type="text" color="yellow">**read-only**</hepta-color><hepta-color type="text" color="yellow">: you can only read the information others expose, and can never modify their resources through this bridge.</hepta-color>
24	
25	- <hepta-color type="text" color="yellow">It relies on </hepta-color><hepta-color type="text" color="yellow">**backend configuration**</hepta-color><hepta-color type="text" color="yellow">: to read a given state, you must supply the other stack's S3 bucket name and the correct key path.</hepta-color>
26	
27	This means `remote_state`'s configuration must match the target stack's `backend.tf` exactly. If you want to read the `int` environment's VPC, your `remote_state` must point to the `int/ap-northeast-1/aws/vpc/terraform.tfstate` path.
28	
29	## outputs.tf: A Stack's Public API and Design Contract
30	
31	In earlier lessons, we treated `outputs.tf` as a small utility for printing information to the terminal. But in a multi-stack architecture, its role changes in kind: **`outputs.tf`**** is this stack's "public API"**.
32	
33	<hepta-color type="text" color="yellow">Only what is explicitly defined in </hepta-color><hepta-color type="text" color="yellow">`outputs.tf`</hepta-color><hepta-color type="text" color="yellow"> gets serialized into the state file, where other stacks can then read it via </hepta-color><hepta-color type="text" color="yellow">`remote_state`</hepta-color><hepta-color type="text" color="yellow">. If a variable does not appear in the outputs, it is invisible to the outside world.</hepta-color>
34	
35	This introduces the **principle of least privilege**:
36	
37	- **Don't expose the entire resource**: although you could output `output "vpc" { value = aws_vpc.main }` to hand over the whole object, that exposes a lot of unnecessary detail.
38	
39	- **Only output the necessary IDs or ARNs**: the best practice is to output only `vpc_id`, `subnet_ids`, `private_route_table_ids` — the information downstream actually needs. This mirrors encapsulation in software engineering: hide the internal implementation and expose only a stable interface.


## Note 40: Module Split Strategy: Why a single giant State is an anti-pattern

- id: `14317547-e8a0-4bba-8344-7d78a8ed2917`
- last_edited: 2026-10-31T07:17:02.513Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Module Split Strategy: Why a single giant State is an anti-pattern" [14317547-e8a0-4bba-8344-7d78a8ed2917] 23 lines
--- Databases ---
none
1	# Module Split Strategy: Why a single giant State is an anti-pattern
2	
3	Earlier we learned that **State is Terraform's memory**: it maps code to real resources.  For safety and collaboration we store it on S3 and lock with DynamoDB. Choosing where to store it is not enough; how you store it matters too.
4	
5	<hepta-color type="text" color="yellow">Early in a project, developers often dump VPC, databases, EKS, even IAM into one directory. That is a monolithic State. In production it is a serious </hepta-color><hepta-color type="text" color="yellow">**anti-pattern**</hepta-color><hepta-color type="text" color="yellow">。 </hepta-color><hepta-color type="text" color="yellow"> It causes four fatal problems:</hepta-color>
6	
7	### 1. Slow runs (Plan/Apply crawls)
8	
9	When you run `terraform plan`, <hepta-color type="text" color="yellow">Terraform must:</hepta-color><hepta-color type="text" color="yellow">**Refresh**</hepta-color><hepta-color type="text" color="yellow">. Using State, it asks the AWS API for the current status of every resource. If one State holds 200 resources, that is at least 200 API calls. As infrastructure grows, seconds become minutes, and AWS may rate-limit you. You only wanted to change a Tag, but you sit five minutes waiting for plan. That kills velocity.</hepta-color>
10	
11	### 2. Huge blast radius
12	
13	This is the most dangerous point. Blast radius is how much is hurt when a change goes wrong. <hepta-color type="text" color="yellow">In a single State, every resource shares one logical boundary. If you mistype a variable while editing an edge Security Group rule, Terraform may decide to destroy and recreate resources—and after you press </hepta-color><hepta-color type="text" color="yellow">`apply`</hepta-color><hepta-color type="text" color="yellow"> you may find it also deleted the production database or VPC.</hepta-color> In one State, a tiny error can take down the whole system.
14	
15	### 3. Mismatched change frequency
16	
17	Cloud resources have very different lifecycles. VPC, subnet layout, and Route53 hosted zones often sit unchanged for months or a year. Kubernetes service config and app-specific IAM Roles may change daily.  Mix them and every frequent app change risks the stable foundation. It is like moving the house foundation every time you change the curtains.
18	
19	### 4. Hard permission control
20	
21	Teams want least privilege. Junior developers should update app config, not change the network or delete KMS keys. If everything is one State, it is hard to split fine-grained IAM read/write on that single S3 State file.
22	
23	<hepta-mention type="card" id="c7fe4dd6-4ab7-471c-8292-6a36f447bc4b">Split strategy: layer by change frequency and ownership</hepta-mention>


## Note 41: Trend Micro Home Page

- id: `75bb5ce5-2067-45b6-9fa1-c8b72a093240`
- last_edited: 2026-10-29T02:44:58.638Z
- whiteboards: Work

card "Trend Micro Home Page" [75bb5ce5-2067-45b6-9fa1-c8b72a093240] 110 lines
--- Databases ---
none
1	# Trend Micro Home Page
2	
3	---
4	
5	- **Ghostty Setup**
6	
7	  ```bash
8	  theme = Dracula
9	  font-size = 14
10	  font-family = Google Sans Code
11	  adjust-cell-height = 5%
12	  font-thicken = true
13	  quick-terminal-position = top
14	  quick-terminal-autohide = false
15	  quick-terminal-animation-duration = 0
16	  quick-terminal-size = 60%, 100%
17	  keybind = global:cmd+grave_accent=toggle_quick_terminal
18	  keybind = global:option+grave_accent=toggle_quick_terminal
19	  window-save-state = always
20	  macos-titlebar-style = tabs
21	  cursor-style = bar      
22	  cursor-click-to-move = true
23	  minimum-contrast = 2
24	  unfocused-split-opacity = 0.85
25	  background-opacity = 0.95
26	  cursor-color = #1BD8F5
27	
28	
29	  keybind = cmd+left=esc:b
30	  keybind = cmd+right=esc:f
31	
32	  # Shader transition effect
33	  custom-shader-animation = true
34	  custom-shader = shaders/cursor-shaders/cursor_warp.glsl
35	  custom-shader = shaders/cursor-shaders/ripple_cursor.glsl
36	  custom-shader = shaders/cursor-shaders/cursor_tail.glsl
37	
38	
39	  ```
40	
41	- **Migrate vscode/cursor extension workflow**
42	
43	  ```go
44	  // macOS 終端機：
45	  code –list-extensions > extensions.txt
46	  //之後在新環境安裝：
47	  xargs -n1 code –install-extension < extensions.txt
48	  ```
49	
50	- **Monitor service account**
51	
52	  ```bash
53	  # db manager username都是: closeloopadmin
54	  # test
55	  password: closeloop112233
56	  # stag
57	  password: closeloop112233stag
58	  # prod
59	  password: closeloop112233prod
60	
61	  # grafana username
62	  admin
63	  cs-grafana
64	  ```
65	
66	---
67	
68	### Working Mindset
69	
70	- **寫 spec 的優先權 (同等套用到 Jira Card) **<hepta-color type="background" color="orange">**橘色共用**</hepta-color>**、**<hepta-color type="background" color="green">**綠色 Jira**</hepta-color>
71	
72	  1. <hepta-color type="background" color="orange">**Background **</hepta-color>
73	
74	  2. <hepta-color type="background" color="green">Definition of Done (DoD)</hepta-color>
75	
76	  3. <hepta-color type="background" color="green">ETA</hepta-color>
77	
78	  4. Flow
79	
80	  5. code
81	
82	- **Standup Meeting: The content should contain:**
83	
84	  1. What I did last work day
85	
86	  2. What I am planning to do today
87	
88	  3. Are there any aspects where I need support from other team members?
89	
90	- **Every member in this team should have the following skill set:**
91	
92	  1. The ability to estimate the effort of each task in this team
93	
94	  2. The ability to plan how to finish an unclear feature
95	
96	  3. Share the domain know-how between each team member, so that each member could be the backup or share the workload between members
97	
98	- **Each PR should be linked in the Jira ticket comment**
99	
100	  - The following information should be left in the Jira ticket comment
101	
102	    - PR link (for develop ticket)
103	
104	    - Document link (for design ticket)
105	
106	    - Verification after service on Production
107	
108	    - Any information to keep for future reference
109	
110	---


## Note 42: RPC Delivery Semantics

- id: `f42bdfba-78ea-4137-a1a5-386568f17761`
- last_edited: 2026-10-25T08:53:29.744Z
- whiteboards: (not placed)

card "RPC Delivery Semantics" [f42bdfba-78ea-4137-a1a5-386568f17761] 103 lines
--- Databases ---
none
1	# RPC Delivery Semantics
2	
3	> **Source:** [Lecture 2: Go RPC 套件介紹](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=59.239&playbackEnd=64.869&_agentLine=25-33)
4	>
5	> This section of the lecture introduces Go's built-in remote procedure call (RPC) package as a cornerstone for distributed programming, highlighting its simplicity compared to traditional C++ alternatives.
6	
7	Imagine you are writing a banking application. You call a local function `Transfer(from, to, amount)`. If the function returns `true`, you know the money moved. If it returns `false`, it didn't. If the program crashes, the transaction either finished or it didn't happen at all. But what happens when that function is an RPC call over a shaky network? You wait for 10 seconds, and then—**Timeout**.
8	
9	Did the server receive the request and crash *after* moving the money? Did the request get lost in a router before even reaching the server? Or did the server successfully finish the job, but the "Success" reply was dropped on the way back to you? In a distributed system, a timeout is not a "failure to execute"; it is a **failure to know**.
10	
11	## The Fundamental Ambiguity
12	
13	The core challenge of RPC is that the network and the remote server are a "black box" to the client. From the client's perspective, there is a sequence of events required for a successful RPC, and a failure at any stage results in the same observation: a timeout or a network error. [\[1\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=421.69&playbackEnd=428.13&_agentLine=182-195)
14	
15	![A technical diagram illustrating the RPC lifecycle and potential failure points. A Client is on the left, a Server on the right, with a network cloud in between. Four distinct failure points are marked with red 'X' icons. Point 1: The Request message is lost in the network. Point 2: The Server crashes after receiving the request but before executing it. Point 3: The Server crashes immediately after executing the request but before sending the reply. Point 4: The Reply message is lost in the network. A bracket on the client side indicates that to the Client, all four scenarios are indistinguishable and result in a generic 'Timeout' error. The labels are 'Client', 'Server', 'Request', 'Execute', 'Reply', 'Timeout', and 'Failure Points'. Clean textbook illustration, white background, professional muted colors.](heptabase://card/f42bdfba-78ea-4137-a1a5-386568f17761/file/2942e056-80fb-4aba-955d-388af2d6ab95)
16	
17	> *The "Fundamental Ambiguity": To a client, a lost request is indistinguishable from a lost reply or a server crashing mid-execution.*
18	
19	As shown in the diagram, there are at least four scenarios where the client sees a timeout:
20	
21	1. **Request Lost:** The network dropped the packet before it reached the server.
22	
23	2. **Server Crash (Pre-execution):** The server received it but died before it could process the request.
24	
25	3. **Server Crash (Post-execution):** The server processed the request (e.g., updated the database) but died before it could send the reply.
26	
27	4. **Reply Lost:** The server finished the work and sent a reply, but the network dropped it.
28	
29	Because the client cannot distinguish between these, the **RPC delivery semantics** define what guarantee the system provides regarding how many times the server will execute the operation.
30	
31	## At-Least-Once Semantics
32	
33	The simplest way to handle a timeout is to **retry**. If the client doesn't get a reply, it simply sends the request again, and again, until it gets an acknowledgment.
34	
35	This provides **at-least-once** semantics: the server will execute the request *one or more* times.
36	
37	### The Power and Peril of Retries
38	
39	At-least-once is easy to implement and very robust against transient network failures. However, it requires the application to handle the consequences of multiple executions. This leads us to the concept of **Idempotency**.
40	
41	An operation is **idempotent** if executing it multiple times has the same effect as executing it once. 
42	
43	- **Idempotent:** `Set(key, 10)`. No matter how many times you tell the server to set the value to 10, the final state is always 10.
44	
45	- **Non-Idempotent:** `Increment(key, 1)`. If you retry this request three times because of dropped replies, the value might increase by 3 instead of 1.
46	
47	In systems like **MapReduce**, at-least-once is a perfect fit. If a "Map" task fails or the reply is lost, the master simply re-runs the task on another worker. Since Map tasks are purely functional and produce deterministic output files, running them twice is harmless—the second run just overwrites the first.
48	
49	## At-Most-Once Semantics
50	
51	For operations that are not idempotent (like bank transfers or non-deterministic writes), at-least-once is dangerous. We need **at-most-once** semantics: the server ensures it executes the request *zero or one* time, but never more.
52	
53	To achieve this, the server must be able to detect and suppress duplicate requests.
54	
55	### Implementing Duplicate Detection
56	
57	The standard mechanism involves three components:
58	
59	1. **Client IDs:** Every client is assigned a unique identifier.
60	
61	2. **Sequence Numbers (XIDs):** Each request from a specific client has a unique, increasing sequence number.
62	
63	3. **Server-side State:** The server maintains a table (often a map) that tracks the last processed XID for each Client ID and the corresponding result.
64	
65	When a request arrives, the server checks its table:
66	
67	- **New Request:** If the XID is greater than the last seen XID for that client, the server executes it, stores the result, and returns it.
68	
69	- **Duplicate Request:** If the XID matches the last seen XID, the server *skips execution* and simply re-sends the **cached result** from the table.
70	
71	- **Old Request:** If the XID is smaller than the last seen, the server can safely ignore it (or return an error).
72	
73	### Challenges of At-Most-Once
74	
75	While at-most-once is safer, it introduces significant engineering overhead:
76	
77	- **State Bloat:** How long should the server keep the XID table? If you have millions of clients, the table can consume massive amounts of RAM. Real systems often use a "sliding window" of XIDs or delete entries after a period of inactivity.
78	
79	- **Server Crashes:** If the server crashes and its memory is wiped, it forgets which XIDs it has already processed. To truly guarantee at-most-once, the XID table must be persisted to non-volatile storage (disk) or replicated across a cluster.
80	
81	## Exactly-Once: The Holy Grail?
82	
83	Engineers often ask: "Can we just have **exactly-once**?" 
84	
85	In a pure sense, exactly-once is impossible in the face of arbitrary network failure because of the **Two Generals' Problem**. You can never be 100% sure the other side knows that you know that they finished.
86	
87	However, in the context of modern distributed systems like **Raft** or **Kafka**, we provide an "effective" exactly-once. This is usually achieved by combining **at-most-once** logic (duplicate detection) with **strong consensus and persistence**. By replicating the XID table across multiple servers using a protocol like Raft, even if one server crashes, the backup knows that a specific request was already processed, ensuring the "single-server illusion" remains intact.
88	
89	## Go RPC in Practice
90	
91	Go provides a `net/rpc` package that is used extensively in the MIT 6.824 labs. It is a "convenient" tool that handles the serialization of Go structures and the management of network connections. [\[2\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=59.239&playbackEnd=64.869&_agentLine=25-33)
92	
93	However, it is important to understand what Go's RPC does *not* do for you:
94	
95	- **No Automatic Retries:** If a `Call()` returns an error, it is up to the client code to decide whether to retry.
96	
97	- **No Built-in Duplicate Detection:** The server does not automatically remember past requests. If you need at-most-once semantics (which you will for the Raft and Key/Value labs), you must implement the ClientID/XID logic yourself.
98	
99	In the labs, Go's RPC is effectively **"best effort."** It sends the request once. If it gets a reply, great. If not, it returns an error. The responsibility of turning this "best effort" into a robust, fault-tolerant system rests entirely on your implementation of the replication and consistency protocols.
100	
101	## Bridge
102	
103	Understanding RPC semantics is the bridge between writing a single-machine program and a distributed one. Once you accept that "No Reply" is a state of ambiguity rather than a simple failure, you begin to see why distributed systems spend so much effort on state management and idempotency. While we have focused on the theoretical "what" of RPC today, the "how" is equally critical. In the next section, we will look at **Go Concurrency Patterns**, specifically how to use goroutines and channels to handle multiple outgoing RPCs in parallel and how to avoid the common pitfalls—like closure capture bugs—that plague distributed implementations.


## Note 43: Lesson 2 note

- id: `f0f78fb0-fb7f-45b5-9633-24aabc580de7`
- last_edited: 2026-10-25T08:49:06.747Z
- whiteboards: Database and Distributed System

card "Lesson 2 note" [f0f78fb0-fb7f-45b5-9633-24aabc580de7] 119 lines
--- Databases ---
none
1	# Lesson 2 note
2	
3	本課深入探討了 Go 語言在分散式系統中的並發模型，從 goroutine 的運作機制到共享記憶體的競態風險。我們對比了 Mutex 與 Channel 兩種編程哲學，並學習了如何利用 Condition Variable 正確協調執行緒以及避免死鎖。
4	
5	### [Go 與 Goroutine](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/dc566d90-4a66-4a6f-86b0-5cb7c4381155)
6	
7	- **Go 的核心優勢**
8	
9	  - **型別與記憶體安全**：防止 C++ 中常見的記憶體隨機覆蓋（scribbling）問題。
10	
11	  - **GC 與執行緒的結合**：垃圾回收簡化了多執行緒環境下共享物件的生命週期管理，工程師無需手動處理複雜的引用計數。
12	
13	  - **內建 RPC 支援**：標準庫提供簡潔的遠端調用機制，提升分散式開發效率。
14	
15	- **Goroutine 的本質**
16	
17	  - 可視為在同一地址空間運作的輕量執行緒。
18	
19	  - 每個 goroutine 有獨立的程式計數器（PC）、暫存器（Registers）與棧（Stack），但共享堆（Heap）記憶體。
20	
21	- **並發的三大動機**
22	
23	  - **I/O 並發**：分散式系統的主要動機。透過讓執行緒在等待網路回應時阻塞，來重疊（overlap）延遲，提升整體吞吐量。
24	
25	  - **多核並行**：利用多個 CPU 核心加速計算密集型任務。
26	
27	  - **背景任務**：執行心跳偵測、定時檢查等輔助工作。
28	
29	- **執行緒 vs 事件驅動**
30	
31	  - Go 選擇執行緒模型是為了「編程簡潔性」，允許寫出順序性的邏輯。
32	
33	  - 透過可變大小的棧（2KB 起步）與多工處理，Go 讓執行緒的開銷接近事件驅動系統。
34	
35	### [共享狀態與競態](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/a9e9224b-c546-4259-a00a-02cd767772a8)
36	
37	- $n++$** 的非原子性陷阱**
38	
39	  - 在機器層級分為 Load-Increment-Store 三步驟，多個 goroutine 交錯執行會導致「遺失更新」。
40	
41	- **競態（Race Condition）的定義**
42	
43	  - 同時滿足三個條件：多個執行緒存取同一記憶體、至少一個是寫入、且缺乏同步機制。
44	
45	- **Go Race Detector**
46	
47	  - 動態分析工具，透過 Shadow Memory 追蹤存取行為。
48	
49	  - **侷限性**：只能偵測到「確實執行過」的程式碼路徑，無法保證靜態上的絕對安全。
50	
51	- **保護不變量（Invariants）**
52	
53	  - 加鎖的目標不只是防止變數被同時修改，更是為了確保操作的「原子性」，讓外部觀察者永遠看不到中間不一致的狀態（如銀行轉帳的中間點）。
54	
55	### [Mutex 與 Channel](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/4250ce6a-c97e-4c58-a2ac-beab6230b1d9)
56	
57	- **兩種並發風格**
58	
59	  - **共享記憶體（Mutex）**：保護數據。適用於狀態機內部的複雜狀態管理（如 Raft）。
60	
61	  - **訊息傳遞（Channel）**：轉移數據所有權。適用於任務委派（Producer/Consumer）或高層級同步。
62	
63	- **閉包捕捉（Closure Capture）陷阱**
64	
65	  - 在 `for` 迴圈中直接使用迴圈變數啟動 goroutine 會導致所有執行緒看到最後一個值。
66	
67	  - **修正法**：必須將變數作為參數傳入函式，利用 Pass-by-value 建立副本。
68	
69	- **Channel 的同步特性**
70	
71	  - 預設是無緩衝的「約會」機制，發送與接收端會互相等待，這使其具備強大的同步能力。
72	
73	- **Web Crawler 案例對比**
74	
75	  - Mutex 版：多個 worker 共享 `fetched` map，需頻繁加解鎖。
76	
77	  - Channel 版：Master 獨佔 map 狀態，worker 透過 channel 回報新網址，結構更清潔且免鎖。
78	
79	### [Deadlock 與 Condition Variable](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/f79d4f49-3ba0-405e-b9b6-2fdb6e70aa58)
80	
81	- **死鎖（Deadlock）的預防**
82	
83	  - 核心原因：循環等待。
84	
85	  - **Raft 重要禁忌**：嚴禁在持有鎖的狀態下發起阻塞式的 RPC 呼叫。這會導致兩台伺服器互相等待對方的回覆而永久凍結。
86	
87	- **Condition Variable (****`sync.Cond`****)**
88	
89	  - 解決 Busy-waiting（輪詢）帶來的 CPU 浪費與高延遲。
90	
91	- **標準使用模式**
92	
93	  - **等待端**：`lock -> for !condition { wait } -> work -> unlock`。必須用 `for` 而非 `if` 來處理「虛假喚醒」（Spurious Wakeups）。
94	
95	  - **通知端**：修改狀態後調用 `Broadcast()`。
96	
97	- **`Wait()`**** 的內在機制**
98	
99	  - 原子地執行：釋放鎖 -> 進入休眠。
100	
101	  - 被喚醒時：必須先重新競爭並獲取鎖，才會從 `Wait()` 回傳。
102	
103	## Q&A
104	
105	- **Q:** Goroutine 的 Registers 和 Stack 位於哪裡？
106	
107	  - Stack 位於實體記憶體（RAM）中，而 Registers 位於 CPU 內部。當 goroutine 執行時，其狀態在 CPU 暫存器；上下文切換時，排程器會將暫存器內容拷貝至該 goroutine 在記憶體中的 Stack。
108	
109	- **Q:** 為什麼在迴圈中啟動 goroutine 需要傳參數？
110	
111	  - 因為匿名函數會「捕捉引用」。如果不傳參，所有 goroutine 會共享迴圈變數的同一個地址，導致它們都抓到迴圈結束後的最終值。傳參數會強制 Go 在每一輪建立值的副本（Pass-by-value）。
112	
113	- **Q:** `sync.Cond` 的 `Wait()` 具體如何運作？為什麼要用 `for`？
114	
115	  - `Wait()` 會原子性地放鎖並休眠。喚醒後，goroutine 必須重新爭奪 Mutex 才能繼續。使用 `for` 是因為在「被喚醒」到「拿到鎖」的間隙，狀態可能已經被其他執行緒改掉了。
116	
117	- **Q:** `Wait` 搶的是資源還是 CPU？
118	
119	  - 搶的是 Mutex 的操作權（資源存取權）。當 goroutine 在 `Wait` 時，它會主動釋放 CPU 供他人使用，直到被喚醒並重新拿回鎖。


## Note 44: Deadlock and Condition Variables

- id: `f79d4f49-3ba0-405e-b9b6-2fdb6e70aa58`
- last_edited: 2026-10-25T08:35:04.375Z
- whiteboards: (not placed)

card "Deadlock and Condition Variables" [f79d4f49-3ba0-405e-b9b6-2fdb6e70aa58] 191 lines
(Content has 191 lines. Use offset and limit parameters to read a specific portion.)


## Note 45: Mutex and Channel

- id: `4250ce6a-c97e-4c58-a2ac-beab6230b1d9`
- last_edited: 2026-10-25T08:24:05.800Z
- whiteboards: (not placed)

card "Mutex and Channel" [4250ce6a-c97e-4c58-a2ac-beab6230b1d9] 195 lines
(Content has 195 lines. Use offset and limit parameters to read a specific portion.)


## Note 46: Shared State and Race Conditions

- id: `a9e9224b-c546-4259-a00a-02cd767772a8`
- last_edited: 2026-10-25T07:58:58.022Z
- whiteboards: (not placed)

card "Shared State and Race Conditions" [a9e9224b-c546-4259-a00a-02cd767772a8] 125 lines
--- Databases ---
none
1	# Shared State and Race Conditions
2	
3	> **Source:** [Lecture 2: Race conditions — the increment example](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=1259.919&playbackEnd=1264.519&_agentLine=519-600) · [Lecture 2: Race detector — how it works, limitations](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=3947.97&playbackEnd=3953.5699999999997&_agentLine=1506-1596) · [Lecture 5: Mutex patterns — protecting invariants](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/014f8697-6e9c-4023-94b8-c6903eb0896a?playbackStart=915.29&playbackEnd=921.67&_agentLine=398-620)
4	>
5	> These lectures break down the fundamental danger of shared memory in concurrent programming, using the classic $n++$ anomaly to illustrate how non-atomic operations lead to lost updates. It also introduces the Go race detector as a dynamic analysis tool for uncovering these subtle, non-deterministic bugs.
6	
7	Imagine you are building a high-performance distributed counter. You spawn a thousand goroutines, each responsible for incrementing a shared variable `n` exactly once. Mathematically, the result should be 1,000. But when you run the code, you get 947. You run it again, and you get 962. If you are lucky, you might even get 1,000 once or twice, only to have it fail again the moment you show it to a colleague. Why does a operation as simple as `n++` fail to produce a predictable result in a concurrent environment?
8	
9	## The Illusion of Atomicity: The $n++$ Anomaly
10	
11	In the previous section, we saw that goroutines share the same address space. <hepta-color type="text" color="yellow">While each goroutine has its own stack and registers to manage its execution state, they all look at the same heap of memory. </hepta-color><hepta-color type="text" color="yellow">[\[1\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=326.23&playbackEnd=333.3&_agentLine=145-155)</hepta-color><hepta-color type="text" color="yellow"> This sharing is powerful, but it creates a fundamental hazard because operations that look like a single step in your source code are actually multiple steps at the machine level.</hepta-color>
12	
13	Consider the statement `n++`. To the CPU, this is not a single atomic "increment" command. Instead, it is typically broken down into three distinct phases:
14	
15	1. **Load**: The current value of `n` is read from memory and placed into a CPU register.
16	
17	2. **Increment**: The CPU adds 1 to the value currently held in that register.
18	
19	3. **Store**: The new value from the register is written back to the memory location for `n`. [\[2\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=1379.049&playbackEnd=1383.869&_agentLine=573-579)
20	
21	The problem arises when two goroutines execute these steps concurrently. If their executions interleave, we get a "lost update."
22	
23	### A Step-by-Step Breakdown of Failure
24	
25	Assume `n` starts at 0. Two goroutines (T1 and T2) both try to execute `n++`:
26	
27	- **Step 1 (T1)**: Loads `n` (0) into its register.
28	
29	- **Step 2 (T2)**: Loads `n` (0) into its own register. *Note: T2 hasn't seen any update yet because T1 hasn't stored anything.*
30	
31	- **Step 3 (T1)**: Increments its register to 1.
32	
33	- **Step 4 (T2)**: Increments its register to 1.
34	
35	- **Step 5 (T1)**: Stores 1 back to `n`.
36	
37	- **Step 6 (T2)**: Stores 1 back to `n`.
38	
39	Final value of `n`: **1**. Even though two increments occurred, one was completely overwritten. [\[3\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=1405.929&playbackEnd=1413.96&_agentLine=582-589) This is the core of the **concurrency hazard**: the physical location of registers and the non-atomic nature of memory access allow for inconsistent views of the system state.
40	
41	## <hepta-color type="text" color="yellow">Defining the Race Condition</hepta-color>
42	
43	<hepta-color type="text" color="yellow">We can now formalize what exactly constitutes a </hepta-color><hepta-color type="text" color="yellow">**Race Condition**</hepta-color><hepta-color type="text" color="yellow">. A race occurs whenever the following three conditions are met simultaneously:</hepta-color>
44	
45	1. <hepta-color type="text" color="yellow">Two or more goroutines access the </hepta-color><hepta-color type="text" color="yellow">**same memory location**</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
46	
47	2. <hepta-color type="text" color="yellow">At least one of these accesses is a </hepta-color><hepta-color type="text" color="yellow">**write**</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
48	
49	3. <hepta-color type="text" color="yellow">There is no </hepta-color><hepta-color type="text" color="yellow">**synchronization (like a lock or channel)**</hepta-color><hepta-color type="text" color="yellow"> used to order the accesses.</hepta-color> [\[4\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=1512.74&playbackEnd=1520.48&_agentLine=624-644)
50	
51	If these conditions exist, the outcome of the program becomes dependent on the "race" between threads—who gets to the store instruction first? Because the operating system and Go runtime can pause and resume goroutines at any millisecond, the exact timing is non-deterministic.
52	
53	### The "Sneaky" Nature of Races
54	
55	Race conditions are often called **Heisenbugs** because they are notoriously difficult to find and reproduce. They might only trigger under specific conditions:
56	
57	- **Low Probability**: As we saw in the lecture demonstration, a race might only fail once in a thousand runs because the window of time for interleaving is just a few nanoseconds. [\[5\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=3889.129&playbackEnd=3892.4889999999996&_agentLine=1481-1492)
58	
59	- **Hardware Sensitivity**: A race that never appears on your dual-core laptop might crash instantly on a 64-core production server where true parallelism increases the chances of collision.
60	
61	- **Load Dependency**: Races often hide until the system is under heavy load, which changes the timing of thread scheduling.
62	
63	The "it usually works" illusion is the most dangerous part of distributed systems engineering. At scale, a "one-in-a-million" bug happens every few minutes. [\[6\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=3923.519&playbackEnd=3930.5&_agentLine=1495-1497)
64	
65	## The Go Race Detector
66	
67	<hepta-color type="text" color="yellow">Because human eyes are terrible at spotting these timing windows, Go provides a powerful tool: the </hepta-color><hepta-color type="text" color="yellow">**Race Detector**</hepta-color><hepta-color type="text" color="yellow">. By adding the </hepta-color><hepta-color type="text" color="yellow">`-race`</hepta-color><hepta-color type="text" color="yellow"> flag (e.g., </hepta-color><hepta-color type="text" color="yellow">`go run -race crawler.go`</hepta-color><hepta-color type="text" color="yellow">), the compiler instruments your code to watch for concurrent memory violations.</hepta-color> [\[7\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=3947.97&playbackEnd=3953.5699999999997&_agentLine=1506-1514)
68	
69	### How It Works: Shadow Memory
70	
71	Internally, the race detector allocates "shadow memory." For every memory location your program uses, the detector keeps a small record in shadow memory tracking:
72	
73	- Which thread most recently read or wrote to that location.
74	
75	- The stack trace of that access.
76	
77	- Any locks held at the time. [\[8\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=4025.34&playbackEnd=4031.81&_agentLine=1537-1548)
78	
79	If a thread writes to a location while another thread is reading it (without an intervening lock release/acquire), the detector intercepts this and immediately prints a detailed report, even if the race didn't actually cause a crash *this time*. 
80	
81	### Critical Limitations
82	
83	<hepta-color type="text" color="yellow">It is vital to understand that the Go race detector is a </hepta-color><hepta-color type="text" color="yellow">**dynamic analysis tool**</hepta-color><hepta-color type="text" color="yellow">, not a static one.</hepta-color>
84	
85	- <hepta-color type="text" color="yellow">**It only catches what runs**</hepta-color><hepta-color type="text" color="yellow">: It does not look at your source code and "prove" it is race-free. It only detects a race if the racing code path is actually executed during your test run. </hepta-color><hepta-color type="text" color="yellow">[\[9\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b69a7be1-1872-498f-86b2-fe693ea89c3b?playbackStart=4092.17&playbackEnd=4098.889999999999&_agentLine=1559-1574)</hepta-color>
86	
87	- <hepta-color type="text" color="yellow">**Performance Overhead**</hepta-color><hepta-color type="text" color="yellow">: It significantly increases memory usage and CPU time (often 2x-10x). You should use it during testing and CI, but rarely in production.</hepta-color>
88	
89	## Protecting Invariants, Not Just Variables
90	
91	As an engineer, you might be tempted to think, "I'll just make every variable atomic." But races are often more subtle than a simple counter. They are about **invariants**—properties of your data that must always remain true.
92	
93	In the "Bank Transfer" example from the lecture, even if we lock individual updates to Alice and Bob's balances, an "audit" thread could still read the state *between* the decrement and the increment. [\[10\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/014f8697-6e9c-4023-94b8-c6903eb0896a?playbackStart=1203.01&playbackEnd=1210.83&_agentLine=526-546) The audit would see a "missing" dollar, even though no money was actually lost.
94	
95	The goal of synchronization is to make these multi-step <hepta-color type="text" color="yellow">operations look </hepta-color><hepta-color type="text" color="yellow">**atomic**</hepta-color><hepta-color type="text" color="yellow"> to the rest of the world, ensuring that no other thread can observe a state where the invariant (e.g., </hepta-color><hepta-color type="text" color="yellow">`Total = Alice + Bob`</hepta-color><hepta-color type="text" color="yellow">) is broken</hepta-color>. [\[11\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/014f8697-6e9c-4023-94b8-c6903eb0896a?playbackStart=1336.65&playbackEnd=1345.22&_agentLine=588-598)
96	
97	## 💡 Checking Your Understanding
98	
99	1. **The Interleaving Exercise**: If three goroutines all run `n++` concurrently and the initial value is 0, what is the *minimum* possible final value of `n`? (Hint: Think about how many updates can be lost if they all load the same value before anyone stores).
100	
101	2. **Detection Limits**: If you have a race condition in a branch of code that only executes when a specific error occurs, will `go test -race` find it during a normal successful test run? Why or why not?
102	
103	3. **Mechanical Sympathy**: Why is it that `n++` is non-atomic, but some 32-bit stores are considered atomic by certain CPUs? How does this relate to the difference between a register and the RAM?
104	
105	## <hepta-color type="text" color="yellow">Connecting to the Solution</hepta-color>
106	
107	<hepta-color type="text" color="yellow">Now that we understand how shared state leads to the "chaos" of race conditions, we need tools to impose order. We have two primary philosophies in Go:</hepta-color>
108	
109	1. <hepta-color type="text" color="yellow">**Shared Memory + Mutexes**</hepta-color><hepta-color type="text" color="yellow">: "Protect the data." We use locks to ensure only one goroutine can touch the data at a time, making complex updates appear atomic.</hepta-color>
110	
111	2. <hepta-color type="text" color="yellow">**Channels**</hepta-color><hepta-color type="text" color="yellow">: "Communicate to avoid sharing." Instead of having multiple goroutines reach for the same variable, we pass the data from one to another.</hepta-color>
112	
113	In the next part, we will see how to apply these solutions to a real-world problem: building a concurrent Web Crawler that doesn't fetch the same page twice.
114	
115	### Key Takeaways
116	
117	- **Goroutines share memory**: This is efficient but dangerous without synchronization.
118	
119	- **Simple operations are not atomic**: `n++` is a three-step process (Load-Increment-Store) that can be interleaved by other threads.
120	
121	- **Race Conditions** require simultaneous access, a write operation, and a lack of synchronization.
122	
123	- **Use the Race Detector**: It is your best defense, but remember it only catches races that actually occur during execution.
124	
125	- **Protect Invariants**: Locking is not just about stopping "lost writes"; it's about ensuring other threads never see a partially updated, inconsistent state. [\[12\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/014f8697-6e9c-4023-94b8-c6903eb0896a?playbackStart=1382.299&playbackEnd=1390.779&_agentLine=611-620)


## Note 47: Go and Goroutines

- id: `dc566d90-4a66-4a6f-86b0-5cb7c4381155`
- last_edited: 2026-10-25T07:41:55.263Z
- whiteboards: (not placed)

card "Go and Goroutines" [dc566d90-4a66-4a6f-86b0-5cb7c4381155] 106 lines
(Content has 106 lines. Use offset and limit parameters to read a specific portion.)


## Note 48: Lesson 1 note

- id: `4c01427c-7710-4da8-b575-a5b0bd714f3f`
- last_edited: 2026-10-25T03:59:25.050Z
- whiteboards: Database and Distributed System

card "Lesson 1 note" [4c01427c-7710-4da8-b575-a5b0bd714f3f] 71 lines
--- Databases ---
none
1	# Lesson 1 note
2	
3	本堂課介紹了 MIT 6.824 課程的全局地圖，並深入探討分散式系統存在的必要性（並行性、容錯、物理需求）及其核心難題：局部失效（Partial Failure）與一致性（Consistency）的權衡，最後透過 CAP 定理正式定義了在網路分區時系統必須做出的設計取捨。
4	
5	### [Course Roadmap](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/7c58a56f-f2e7-4b4d-bc27-1aabc43575bb)
6	
7	- 課程旨在將學生從分散式系統的「使用者」轉變為「設計者」，學習如何從第一原則（First Principles）出發設計可靠的後端架構。
8	
9	- 學習路徑分為四大階段：大規模儲存（從 GFS 開始理解單點瓶頸）、共識演算法（以 Raft 為核心解決機器群的協議問題）、交易與一致性（探討 Spanner 等全球規模的 ACID 保證）、以及開放式系統（Bitcoin 等不信任環境下的共識）。
10	
11	- 核心目標在於建立閱讀原始論文並分析技術取捨（Trade-offs）的能力，而非僅僅是黑盒式的架構模仿。
12	
13	### [Why Distribute at All](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/546714a5-78dc-4493-8039-7fa65642e1a3)
14	
15	- 採用分散式系統的三大驅動力：
16	
17	  - 並行性（Parallelism）：透過增加機器實現可擴展的加速（Scalable Speedup），但會遇到「擴展天花板」（Scalability Ceiling），即效能瓶頸轉移至共享狀態（如資料庫）。
18	
19	  - 容錯（Fault Tolerance）：在成千上萬台機器隨時可能有組件損壞的現實下，確保系統可用性與可恢復性。
20	
21	  - 物理必要性（Physical Necessity）：資料天生分散於全球各地，且具備安全隔離的需求。
22	
23	- 分散式系統的本質困難在於「局部失效」（Partial Failure）：網路可能不穩定、伺服器可能變慢或假死，這種不確定性是所有複雜度的根源。
24	
25	- 應對工具：使用非揮發性儲存（Non-volatile Storage）來恢復狀態，以及使用副本（Replication）來提供容錯，但後者會引發一致性問題。
26	
27	### [Replication's Hidden Cost](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/98b25abe-4121-4ce4-8316-8fa5cec1bff0)
28	
29	- 「副本循環」：為了效能進行分片（Sharding）→ 為了容錯引入副本（Replication）→ 副本之間資料不同步引發一致性挑戰。
30	
31	- 強一致性（Strong Consistency / Linearizability）是開發者的理想模型：系統表現得像單機一樣，讀取必須回傳最近一次寫入的結果。
32	
33	- 強一致性的昂貴代價：
34	
35	  - 通訊開銷：副本之間需要大量「閒聊」（Chitchat）來確認狀態，造成網路瓶頸。
36	
37	  - 物理限制：地理位置分散的系統受限於光速延遲，強一致性會大幅推升回應延遲（Latency）。
38	
39	- 弱一致性（Weak Consistency）的選擇：許多大規模系統為了極致速度與可用性，容許暫時的舊資料（Stale Read）。
40	
41	### [The CAP Theorem](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5d06f418-7369-44bb-8825-1ac58b651c70)
42	
43	- 三大屬性定義：一致性（C, 原子一致性）、可用性（A, 請求必有回應）、分區容忍性（P, 忍受訊息丟失）。
44	
45	- 不可能證明：在非同步網路模型中，一旦發生分區（Partition），系統若要保證一致性就必須拒絕回應（犧牲 A），若要保證可用性就必須回傳可能過時的資料（犧牲 C）。
46	
47	- 實務設計取捨：P 在現代分散式環境中是「現實前提」而非「選項」。
48	
49	  - CP 系統：寧願報錯也不願給錯資料（如使用 Raft 的 Etcd）。
50	
51	  - AP 系統：寧願給舊資料也不願不回應（如 Cassandra）。
52	
53	- 現代折衷方案：利用部分同步（Partially Synchronous）模型與 $t$-Connected Consistency，在網路健康時提供強保證，分區時定義明確的失敗模式。
54	
55	## Q&A
56	
57	- **Q:** 為什麼 P（分區容忍性）在 CAP 中不是一個可以選擇「不要」的選項？
58	
59	  - 分區（Partition）是網路環境的現實條件（如光纜斷裂、GC 暫停），不是設計者能決定的。
60	
61	  - P 的含義是「當分區發生時，系統是否能不崩潰並繼續運作」。
62	
63	  - 因為分區必然會發生，所以設計者真正的抉擇是在分區發生的那一刻，要選擇 CP（保證正確但暫時停服）還是 AP（保證服務但允許資料不一致）。
64	
65	- **Q:** P（分區容忍性）和 A（可用性）感覺很像，具體區別在哪？
66	
67	  - P 是面對「故障」的抗壓性（條件），描述網路狀態是否斷裂。
68	
69	  - A 是對「用戶」的服務承諾（結果），描述節點是否必須給出回應。
70	
71	  - 關鍵區別在於「分區發生時」：CP 系統有 P 但沒 A（為了不給錯資料而拒絕回答）；AP 系統有 P 也有 A（儘管資料不一致，仍堅持要回答用戶）。


## Note 49: The CAP Theorem (現代系統中，P其實是必要的，所以我們只會在CA中做抉擇）

- id: `5d06f418-7369-44bb-8825-1ac58b651c70`
- last_edited: 2026-10-25T03:51:35.416Z
- whiteboards: (not placed)

card "The CAP Theorem (現代系統中，P其實是必要的，所以我們只會在CA中做抉擇）" [5d06f418-7369-44bb-8825-1ac58b651c70] 119 lines
(Content has 119 lines. Use offset and limit parameters to read a specific portion.)


## Note 50: Replication's Hidden Cost

- id: `98b25abe-4121-4ce4-8316-8fa5cec1bff0`
- last_edited: 2026-10-25T03:31:40.005Z
- whiteboards: (not placed)

card "Replication's Hidden Cost" [98b25abe-4121-4ce4-8316-8fa5cec1bff0] 93 lines
--- Databases ---
none
1	# Replication's Hidden Cost
2	
3	> **Source:** [Lecture 1: Introduction — consistency, strong vs weak, replication cost](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2224.12&playbackEnd=2233.2&_agentLine=1072-1320)
4	>
5	> This lecture segment explores how the necessity of replication for fault tolerance inevitably leads to the "consistency problem," illustrating through a client-crash scenario how multiple replicas can easily diverge and return conflicting data.
6	
7	If you have a single server and you need to handle more traffic, the obvious solution is to add another server. If you need to ensure your data survives a disk failure, the obvious solution is to make a copy of that data on another machine. In distributed systems, "more" usually seems like the answer. But there is a hidden tax on this "more." 
8	
9	Why is it that the very mechanism we use to make systems reliable—replication—is the exact same thing that makes them "lie" to us? We often start by trying to make things faster or more reliable, but we almost always end up in the same place: staring at a **consistency problem**.
10	
11	## <hepta-color type="text" color="yellow">The Replication Loop</hepta-color>
12	
13	<hepta-color type="text" color="yellow">To understand why we struggle with consistency, we have to look at the logical chain of events that drives modern system architecture. Most engineers don't set out to build a complex distributed system; they are forced into it by a recurring cycle of requirements.</hepta-color>
14	
15	1. <hepta-color type="text" color="yellow">**High Performance → Sharding:**</hepta-color><hepta-color type="text" color="yellow"> When a single machine can no longer handle the request volume or the dataset size, we split the data across multiple machines. This is </hepta-color><hepta-color type="text" color="yellow">**sharding**</hepta-color><hepta-color type="text" color="yellow"> (or partitioning). </hepta-color><hepta-color type="text" color="yellow">[\[1\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2319.36&playbackEnd=2326.1499999999996&_agentLine=1121-1124)</hepta-color>
16	
17	2. <hepta-color type="text" color="yellow">**Multiple Servers → Faults:**</hepta-color><hepta-color type="text" color="yellow"> As soon as you move from one server to ten (or a thousand), the probability that </hepta-color><hepta-color type="text" color="yellow">*something*</hepta-color><hepta-color type="text" color="yellow"> is broken at any given moment approaches 100%. In a single-machine world, a crash is an all-or-nothing event. In a distributed world, </hepta-color><hepta-color type="text" color="yellow">**partial failure**</hepta-color><hepta-color type="text" color="yellow"> is the norm.</hepta-color>
18	
19	3. <hepta-color type="text" color="yellow">**Fault Tolerance → Replication:**</hepta-color><hepta-color type="text" color="yellow"> To ensure the system stays up when a shard's primary server fails, we create copies of that shard's data on other servers. This is </hepta-color><hepta-color type="text" color="yellow">**replication**</hepta-color><hepta-color type="text" color="yellow">. </hepta-color><hepta-color type="text" color="yellow">[\[1\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2319.36&playbackEnd=2326.1499999999996&_agentLine=1121-1124)</hepta-color>
20	
21	4. <hepta-color type="text" color="yellow">**Replication → The Consistency Problem:**</hepta-color><hepta-color type="text" color="yellow"> Now that you have multiple copies of the same data, you face a terrifying question: </hepta-color><hepta-color type="text" color="yellow">**Which copy is the "truth"?**</hepta-color><hepta-color type="text" color="yellow"> </hepta-color><hepta-color type="text" color="yellow">[\[2\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2356.4&playbackEnd=2362.1499999999996&_agentLine=1137-1143)</hepta-color>
22	
23	<hepta-color type="text" color="yellow">This is the "Replication Loop." Every step is a rational response to a real-world constraint, yet the final result is a system where different servers might tell the client different stories about the same piece of data.</hepta-color>
24	
25	## The Stale-Read Scenario
26	
27	Let's look at how this breaks in practice using a classic "lost write" or "stale read" example. Imagine we have a simple key-value service that supports two operations: `put(key, value)` and `get(key)`. [\[3\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2242.72&playbackEnd=2246.87&_agentLine=1078-1090)
28	
29	Suppose we have two servers, $S1$ and $S2$, each maintaining a copy of the value for key $x$. Initially, both servers agree that $x = 20$. [\[4\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2381.56&playbackEnd=2386.6&_agentLine=1149-1151)
30	
31	Now, a client wants to update the value to $21$. To keep the replicas in sync, the client must send a `put(x, 21)` to both servers.
32	
33	1. The client sends `put(x, 21)` to $S1$. $S1$ updates its local table.
34	
35	2. **The Disaster:** Just before the client can send the second request to $S2$, the client's machine crashes (or the network cable is pulled). [\[5\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2416.72&playbackEnd=2423.1499999999996&_agentLine=1163-1167)
36	
37	The system is now in a **divergent state**: $S1$ thinks $x = 21$, but $S2$ still thinks $x = 20$. [\[6\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2428.6&playbackEnd=2434.35&_agentLine=1170-1173)
38	
39	![A sequence diagram showing a client successfully sending a "put(k, 21)" to Server 1 but crashing before it can send the same request to Server 2. A second client then issues a "get(k)" to Server 2 and receives the old value "20". The diagram uses clean lines and a white background, with a "Client Crash" symbol and two separate server boxes showing their internal state tables. Labels include "Server 1: k=21", "Server 2: k=20", "Put(k, 21)", "Get(k)", and "Response: 20 (Stale!)".](heptabase://card/98b25abe-4121-4ce4-8316-8fa5cec1bff0/file/22ea309e-9b2e-47f4-ba0e-581bd5346b87)
40	
41	> *In this scenario, a partial failure during the write process leaves the replicas inconsistent. A subsequent reader might see the new value or the old value depending on which replica they happen to query.* [\[7\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2434.36&playbackEnd=2440.83&_agentLine=1174-1178)
42	
43	Even if you have a rule that says "always talk to the primary," if the primary ($S1$) fails later, the system will failover to the backup ($S2$). If the backup hasn't seen the update, your system will suddenly "go back in time," serving a value that was supposedly overwritten a week ago. [\[8\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2449.4&playbackEnd=2457.1499999999996&_agentLine=1181-1194)
44	
45	## Strong Consistency: The Gold Standard
46	
47	To prevent these "time-travel" anomalies, we need a formal contract between the system and the developer. This contract is what we call a **Consistency Model**. [\[9\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2485.6&playbackEnd=2492.2299999999996&_agentLine=1202-1208)
48	
49	The most intuitive model—and the one that application developers desperately want—is **Strong Consistency** (often used interchangeably with **Linearizability** in this course).
50	
51	A strongly consistent system provides a single-server illusion. Even though there are dozens of replicas under the hood, the system behaves as if there were only one copy of the data. Specifically:
52	
53	> A `get(key)` must return the value of the most recently completed `put(key, value)`. [\[10\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2507.32&playbackEnd=2512.47&_agentLine=1212-1217)
54	
55	If a system is strongly consistent, the "21 vs 20" problem disappears. The system would ensure that either the `put` is seen by *everyone* before it's considered "complete," or the `get` would be forced to wait or coordinate to find the latest version.
56	
57	## The High Cost of the Ideal
58	
59	If strong consistency is so great, why don't we use it for everything? The answer is simple: **Coordination is slow.** [\[11\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2579.12&playbackEnd=2586.2299999999996&_agentLine=1246-1250)
60	
61	To guarantee that a `get` sees the most recent `put`, either the writer or the reader (or both) must talk to enough replicas to ensure they haven't missed anything. [\[12\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2592.28&playbackEnd=2599.27&_agentLine=1253-1255) This "chitchat" creates two massive problems:
62	
63	### 1. The Network Bottleneck
64	
65	Every request now requires multiple internal RPCs (Remote Procedure Calls) to other servers. Instead of one round-trip from the client to the server, you now have a web of communication between replicas. This dramatically increases the load on the network and the latency of every single operation. [\[13\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2620.92&playbackEnd=2624.91&_agentLine=1266-1268)
66	
67	### 2. The Speed of Light (Physical Necessity)
68	
69	To get the best fault tolerance, you want your replicas to be geographically separated. You want one in New York and one in London so that a hurricane in one doesn't kill both. [\[14\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2665.32&playbackEnd=2670.6699999999996&_agentLine=1285-1308)
70	
71	However, the speed of light is a hard limit. A round-trip between cross-continental data centers takes tens of milliseconds. [\[15\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2716.36&playbackEnd=2722.75&_agentLine=1311-1319) In computer time, 30 milliseconds is an eternity—your CPU could have executed tens of millions of instructions while waiting for that "Strong Consistency" confirmation from across the ocean. [\[16\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2760.28&playbackEnd=2767.35&_agentLine=1334-1339)
72	
73	## <hepta-color type="text" color="yellow">Weak Consistency as a Choice</hepta-color>
74	
75	<hepta-color type="text" color="yellow">Because strong consistency is so expensive, many real-world systems (like Amazon's early Dynamo or various caching layers) choose </hepta-color><hepta-color type="text" color="yellow">**Weak Consistency**</hepta-color><hepta-color type="text" color="yellow">. They allow stale reads—meaning a </hepta-color><hepta-color type="text" color="yellow">`get`</hepta-color><hepta-color type="text" color="yellow"> might return an old value—in exchange for incredible speed and availability. </hepta-color><hepta-color type="text" color="yellow">[\[17\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2624.92&playbackEnd=2631.27&_agentLine=1268-1273)</hepta-color>
76	
77	<hepta-color type="text" color="yellow">In a weakly consistent system, a server can respond to a </hepta-color><hepta-color type="text" color="yellow">`get`</hepta-color><hepta-color type="text" color="yellow"> using only its local, potentially outdated data. It doesn't have to wait to talk to its neighbors. This is much faster, but it places the burden of handling "lying" data on the application developer.</hepta-color>
78	
79	Choosing between these models isn't just about technical preference; it's a fundamental trade-off. As we will see in the next part, this trade-off is eventually codified into a formal law of distributed systems: the **CAP Theorem**.
80	
81	### The Road to the Impossible
82	
83	We have seen that replication is necessary for safety but introduces the risk of divergence. We have seen that fixing that divergence (Strong Consistency) requires heavy coordination that kills performance. This tension between "getting the right answer" and "getting any answer quickly" is the central conflict of the course. In the next section, we will see how researchers formally proved that you can't have your cake and eat it too when the network breaks.
84	
85	**Summary of the Replication Challenge**
86	
87	- **Replication** is required for fault tolerance but creates multiple versions of the truth.
88	
89	- **Consistency** is the set of rules that tells us which version a client will see.
90	
91	- **Strong Consistency** mimics a single server but requires expensive cross-replica communication.
92	
93	- **Latency** (often dictated by the speed of light) makes strong consistency impractical for many global-scale systems.


## Note 51: Why Distribute at All → 分散式所帶來的好處以及相應的 Complexity (Concurrency Bug, Consistency)

- id: `546714a5-78dc-4493-8039-7fa65642e1a3`
- last_edited: 2026-10-25T03:16:09.607Z
- whiteboards: (not placed)

card "Why Distribute at All → 分散式所帶來的好處以及相應的 Complexity (Concurrency Bug, Consistency)" [546714a5-78dc-4493-8039-7fa65642e1a3] 71 lines
--- Databases ---
none
1	# Why Distribute at All → 分散式所帶來的好處以及相應的 Complexity (Concurrency Bug, Consistency)
2	
3	> **Source:** [Lecture 1: Introduction](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=64.839&playbackEnd=71.94999999999999&_agentLine=33-1050)
4	>
5	> This lecture introduces the core motivations for distributed systems—parallelism, fault tolerance, and physical necessity—while highlighting the architectural challenges posed by partial failures and the scalability ceiling.
6	
7	If you were given a choice between hiring ten world-class engineers to optimize your code or simply spending \$10,000 on more cloud instances to handle your traffic, which would you choose? Most businesses choose the machines. In the world of infrastructure, we call this the "huge hammer": the ability to solve performance problems with money rather than scarce engineering time. But as you will soon discover, this hammer comes with a hidden "complexity tax" that can break even the most robust designs.
8	
9	## The Three Drivers of Distribution
10	
11	Before we touch a single line of Go code or look at a consensus protocol, we must ask why we are doing this at all. Distributed systems are significantly harder to build, debug, and maintain than single-machine programs. As Robert Morris notes, if you can solve your problem on one computer, you should. [\[1\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=64.839&playbackEnd=71.94999999999999&_agentLine=33-45) <hepta-color type="text" color="yellow">We only move to multiple cooperating computers when forced by three primary drivers.</hepta-color>
12	
13	### <hepta-color type="text" color="yellow">1. Parallelism (Scalable Speedup)</hepta-color>
14	
15	<hepta-color type="text" color="yellow">The most common motivation is the desire for </hepta-color><hepta-color type="text" color="yellow">**scalable speedup**</hepta-color><hepta-color type="text" color="yellow">. Ideally, if you double the number of computers, you should be able to handle twice the throughput or finish a task in half the time.</hepta-color> [\[2\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=96.96&playbackEnd=104.03&_agentLine=53-57)
16	
17	This is the "huge hammer" of distributed systems. Buying 1,000 computers is a purely financial decision, whereas paying programmers to restructure software for extreme efficiency is expensive and time-consuming. [\[3\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=1440.68&playbackEnd=1447.83&_agentLine=691-716) However, achieving this 1,000x speedup is rarely a linear process. As we add more workers, we often run into the **scalability ceiling**.
18	
19	Imagine a standard web architecture: users talk to a pool of web servers, which in turn talk to a single database. Initially, adding more web servers works perfectly. But eventually, the database becomes the bottleneck. Every additional web server you add at that point doesn't help—it might even hurt by increasing contention on the database. [\[4\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=1644.6&playbackEnd=1652.59&_agentLine=792-808)
20	
21	![A technical diagram illustrating the 'Scalability Ceiling' in a web application architecture. On the left, several web server icons are shown behind a load balancer, with arrows indicating incoming requests. As more web servers are added, they all point to a single, large database server icon on the right. The database server is highlighted in red with a warning pattern, showing many overlapping arrows creating a visual bottleneck. A graph above the architecture shows a throughput curve that initially rises linearly with the number of servers but then flattens out as the database reaches its limit. Labels include 'Load Balancer', 'Web Servers', 'Database Bottleneck', and 'Throughput Limit'. Clean textbook illustration style, white background, muted colors.](heptabase://card/546714a5-78dc-4493-8039-7fa65642e1a3/file/a9fd2b55-c2e1-4004-9147-a780a263c05f)
22	
23	> <hepta-color type="text" color="yellow">*The scalability ceiling: Adding more stateless web servers eventually moves the bottleneck to the shared state—the database.*</hepta-color> [\[4\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=1644.6&playbackEnd=1652.59&_agentLine=792-808)
24	
25	### <hepta-color type="text" color="yellow">2. Fault Tolerance</hepta-color>
26	
27	<hepta-color type="text" color="yellow">In a single-machine world, failures are rare. A server might stay up for years without a crash. </hepta-color><hepta-color type="text" color="yellow">[\[5\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=1763.16&playbackEnd=1771.07&_agentLine=843-854)</hepta-color><hepta-color type="text" color="yellow"> But in a system with 1,000 machines, a 99.9% uptime per machine means you can expect roughly three failures every day. </hepta-color><hepta-color type="text" color="yellow">[\[6\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=1786.32&playbackEnd=1793.19&_agentLine=855-862)</hepta-color>
28	
29	<hepta-color type="text" color="yellow">Distribution allows us to build </hepta-color><hepta-color type="text" color="yellow">**available**</hepta-color><hepta-color type="text" color="yellow"> systems—those that keep operating correctly despite individual component failures. It also forces us to consider </hepta-color><hepta-color type="text" color="yellow">**recoverability**</hepta-color><hepta-color type="text" color="yellow">: the ability of a system to stop responding during a major failure but resume exactly where it left off once repaired, without losing data.</hepta-color> [\[7\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=1907.36&playbackEnd=1914.36&_agentLine=920-979)
30	
31	### 3. Physical Necessity and Security
32	
33	Some problems are distributed by their very nature. If you want to transfer money from a bank in New York to a bank in London, you have no choice but to deal with two computers separated by an ocean. [\[8\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=131.24&playbackEnd=138.15&_agentLine=65-76)
34	
35	Security is the final driver. By splitting a computation across computers owned by different parties and communicating via narrow network protocols, you can isolate untrusted code and minimize the blast radius of a malicious actor. [\[9\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=155.64&playbackEnd=162.51000000000002&_agentLine=79-99)
36	
37	## The Reality of Partial Failures
38	
39	What makes distributed systems fundamentally harder than single-machine systems is the concept of **partial failure**.
40	
41	When a single computer fails, it usually just stops. It either works or it crashes—a "fail-stop" model. But a distributed system is made of many moving parts and a network. You can have a scenario where the computer is perfectly healthy, but the network cable connecting it to the database is unreliable. Or a switch might overheat, causing some packets to drop while others pass through. [\[10\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=236.84&playbackEnd=245.47&_agentLine=122-138)
42	
43	In a large-scale system, there is **always something broken**. [\[11\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=1849.6&playbackEnd=1856.75&_agentLine=888-898) This constant state of partial failure means that your code cannot simply assume that an RPC (Remote Procedure Call) will either succeed or return a "server down" error. It might time out. Did the request get to the server? Did the server execute it and then crash? Or is the server just being very slow? This ambiguity is the root of the complexity we will spend this course solving.
44	
45	## <hepta-color type="text" color="yellow">Our Tools: Storage and Replication</hepta-color>
46	
47	<hepta-color type="text" color="yellow">To fight back against this chaos, we rely on two primary tools:</hepta-color>
48	
49	1. <hepta-color type="text" color="yellow">**Non-volatile Storage**</hepta-color><hepta-color type="text" color="yellow">: Using hard drives or SSDs to store checkpoints or logs. This ensures that when a system eventually recovers from a power failure, it can read its state and continue. However, updating non-volatile storage is slow and expensive. </hepta-color><hepta-color type="text" color="yellow">[\[12\]](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/5aee26e8-a313-4c7f-96a7-109f701a5eb8?playbackStart=2087.48&playbackEnd=2092.47&_agentLine=1008-1025)</hepta-color>
50	
51	2. <hepta-color type="text" color="yellow">**Replication**</hepta-color><hepta-color type="text" color="yellow">: Keeping multiple copies of the data across different machines. If one replica dies, the others can take over instantly.</hepta-color>
52	
53	While these tools provide the foundation for fault tolerance, they introduce a massive new problem: **consistency**. If you have three replicas and you write a "1" to the first one but it crashes before telling the others, what should a reader see?
54	
55	## The Complexity Tax
56	
57	The takeaway for any senior engineer is this: **Distributed systems are a means to an end, not an end in themselves.** You should avoid them until you hit a wall that only distribution can climb—whether that's a performance bottleneck, a requirement for 24/7 availability, or a physical separation of data. Once you cross that line, you are no longer just writing logic; you are managing a swarm of unreliable components where the "happy path" is the exception, and partial failure is the rule.
58	
59	### The Complexity of Distribution
60	
61	Each reason for distribution introduces a specific type of complexity that we will explore in the coming weeks:
62	
63	- **Parallelism** leads to **concurrency bugs** (race conditions, deadlocks).
64	
65	- **Fault Tolerance** leads to **consistency issues** (stale data, split-brain).
66	
67	- **Partial Failures** lead to **network protocol complexity** (timeouts, retries, idempotency).
68	
69	## The Path to Consistency
70	
71	We have seen that we distribute systems to gain performance and fault tolerance, and we use replication to achieve those goals. However, as soon as we have more than one copy of our data, we face a fundamental trade-off. In the next part, we will explore "Replication's Hidden Cost" and define what it truly means for a system to be "consistent" when its data is scattered across the globe.


## Note 52: Reading plan output and Apply: four fate symbols

- id: `5dbeaa9c-df31-494a-9e48-6f4f6fbf6103`
- last_edited: 2026-10-23T02:20:43.748Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Reading plan output and Apply: four fate symbols" [5dbeaa9c-df31-494a-9e48-6f4f6fbf6103] 55 lines
--- Databases ---
none
1	# Reading plan output and Apply: four fate symbols
2	
3	When you run `terraform plan`, you see a string of symbols. Reading them is basic platform-engineer skill:
4	
5	### <hepta-color type="text" color="yellow">`+`</hepta-color><hepta-color type="text" color="yellow"> (Create): create a new resource</hepta-color>
6	
7	The resource does not exist in the cloud yet, or is not under current State.
8	
9	- **Example**: you added an `aws_s3_bucket` resource block.
10	
11	- **Note**: some attributes show as `(known after apply)`. Values like Resource ID are only known after AWS actually creates the resource.
12	
13	### <hepta-color type="text" color="yellow">`~`</hepta-color><hepta-color type="text" color="yellow"> (Update in-place): modify in place</hepta-color>
14	
15	The ideal case. The resource stays; only some attributes (tags, description, some non-destructive settings) change.
16	
17	- <hepta-color type="text" color="yellow">**Example**</hepta-color><hepta-color type="text" color="yellow">: changing an EC2 instance tag, or a Security Group rule.</hepta-color>
18	
19	- **Risk**: usually low, but some param changes may restart the service depending on the provider.
20	
21	### <hepta-color type="text" color="yellow">`-`</hepta-color><hepta-color type="text" color="yellow"> (Destroy): delete a resource</hepta-color>
22	
23	The resource was removed from your code, or you reduced `for_each` count.
24	
25	- <hepta-color type="text" color="yellow">**Example**</hepta-color><hepta-color type="text" color="yellow">: deleting an unused test Subnet.</hepta-color>
26	
27	- **Warning**: treat this symbol with care; confirm you really want to delete it.
28	
29	### <hepta-color type="text" color="yellow">`-/+`</hepta-color><hepta-color type="text" color="yellow"> (Replace): destroy then create</hepta-color>
30	
31	<hepta-color type="text" color="yellow">The most dangerous symbol. It usually happens when you change an </hepta-color><hepta-color type="text" color="yellow">**immutable**</hepta-color><hepta-color type="text" color="yellow"> attribute.</hepta-color>
32	
33	- **Example**: in AWS you cannot rename some core identifiers of an existing VPC. If you change `aws_db_instance` `identifier` (database ID), the AWS API does not support rename; it requires delete-old, create-new.
34	
35	- **Impact**: downtime, and without backups, data loss. When you see `-/+`, look closely at which attribute triggered `forces replacement`。
36	
37	## <hepta-color type="text" color="yellow">apply: from human approval to automation</hepta-color>
38	
39	In Terraform's design, `apply` is a controlled approval step, not a casual command.
40	
41	In development (local), we usually run `terraform apply` directly. Terraform re-runs `plan` and waits for you to type `yes`. That is the last human gate before production.
42	
43	1. **Auto-approve**: use `terraform apply -auto-approve`. Typical for int/stg, or changes already reviewed in a PR.
44	
45	2. **Plan-file apply**: first `terraform plan -out=tfplan` saves the preview as a binary file, then at `apply` use `terraform apply tfplan`. What you saw is what you run, so a manual cloud change in the few minutes between plan and apply cannot skew the result.
46	
47	## Helper tools: inspect State
48	
49	Sometimes you do not need the full flow; you just want to see managed resources. These commands help:
50	
51	- `terraform state list`: list every resource path in current State. Confirms whether resources from a given `for_each` actually exist.
52	
53	- `terraform state show <path>`: show every attribute of one resource, including values the cloud generated that are not in your code.
54	
55	- `terraform output`: print every output defined in `outputs.tf`. The most common check when modules talk to each other.c


## Note 53: Core mental model: the triangle comparison

- id: `bf7a68b3-935b-46aa-8084-d6ac1e6581f5`
- last_edited: 2026-10-23T02:19:29.399Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Core mental model: the triangle comparison" [bf7a68b3-935b-46aa-8084-d6ac1e6581f5] 13 lines
--- Databases ---
none
1	# Core mental model: the triangle comparison
2	
3	How can Terraform tell you exactly what will change? A triangle comparison. Unlike tools that only look at script content, Terraform on `plan` watches three dimensions at once:
4	
5	1. **Code (desired state)**: how you want the world to look.
6	
7	2. **State (Memory)**: what Terraform remembered after the last run.
8	
9	3. **Actual resources (reality)**: what really exists in the cloud console.
10	
11	<hepta-color type="text" color="yellow">When you run </hepta-color><hepta-color type="text" color="yellow">`plan`</hepta-color><hepta-color type="text" color="yellow">, Terraform first syncs State with actual resources (Refresh), then compares code to the latest reality.</hepta-color>
12	
13	That is how Terraform detects **configuration drift**. If someone quietly changes a Subnet tag in the AWS Console and your code did not change, `plan` sees reality vs. intent and proposes putting the tag back. Code as the single source of truth is something ClickOps can never match.


## Note 54: Implicit dependency: how Terraform orders work

- id: `d4e8e090-25fa-4fd0-a2f3-870290bdc8c4`
- last_edited: 2026-10-23T02:02:01.982Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Implicit dependency: how Terraform orders work" [d4e8e090-25fa-4fd0-a2f3-870290bdc8c4] 55 lines
--- Databases ---
none
1	# Implicit dependency: how Terraform orders work
2	
3	<hepta-color type="text" color="yellow">How do the four blocks connect? One of Terraform’s nicest traits: </hepta-color><hepta-color type="text" color="yellow">**implicit dependency**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
4	
5	<hepta-color type="text" color="yellow">In a classic script you write </hepta-color><hepta-color type="text" color="yellow">`create_vpc`</hepta-color><hepta-color type="text" color="yellow"> then </hepta-color><hepta-color type="text" color="yellow">`create_subnet`</hepta-color><hepta-color type="text" color="yellow">. In Terraform, order comes from </hepta-color><hepta-color type="text" color="yellow">**references**</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
6	
7	### Why this matters
8	
9	<hepta-color type="text" color="yellow">In the same </hepta-color><hepta-color type="text" color="yellow">**directory (Module)**</hepta-color><hepta-color type="text" color="yellow">, the </hepta-color><hepta-color type="text" color="yellow">**physical order**</hepta-color><hepta-color type="text" color="yellow"> of resource declarations does not matter. Subnet first, VPC last, Terraform still creates VPC first. You can group by function (split files) instead of by execution time.</hepta-color>
10	
11	<hepta-color type="text" color="yellow">If two resources have no direct or indirect reference, Terraform tries to create them </hepta-color><hepta-color type="text" color="yellow">**in parallel**</hepta-color><hepta-color type="text" color="yellow"> to save time.</hepta-color>
12	
13	Look at this combination:
14	
15	```hcl
16	# 1. Define a variable
17	variable "vpc_cidr" { default = "10.0.0.0/16" }
18	
19	# 2. Define a computed constant
20	locals {
21	  vpc_name = "production-vpc"
22	}
23	
24	# 3. Create the VPC resource
25	resource "aws_vpc" "main" {
26	  cidr_block = var.vpc_cidr   # reference variable
27	  tags = {
28	    Name = local.vpc_name     # reference locals
29	  }
30	}
31	
32	# 4. Create the Subnet resource
33	resource "aws_subnet" "private" {
34	  vpc_id     = aws_vpc.main.id   # reference another resource’s attribute!
35	  cidr_block = "10.0.1.0/24"
36	}
37	```
38	
39	Notice?<hepta-color type="text" color="yellow"> In </hepta-color><hepta-color type="text" color="yellow">`aws_subnet`</hepta-color><hepta-color type="text" color="yellow"> we referenced </hepta-color><hepta-color type="text" color="yellow">`aws_vpc.main.id`</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
40	
41	<hepta-color type="text" color="yellow">To Terraform that is not just a value; it is a strong signal: </hepta-color><hepta-color type="text" color="yellow">**“Subnet must wait until VPC is created”**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color><hepta-color type="text" color="yellow">** Terraform builds a dependency graph behind the scenes and places VPC upstream of Subnet.**</hepta-color>
42	
43	## How the four blocks interact
44	
45	Think of a factory:
46	
47	1. **Variable** is raw material (brought in from outside).
48	
49	2. **Locals** is in-factory processing and intermediates (computed internally, not visible outside).
50	
51	3. **Resource** is the finished product (what actually exists in the cloud).
52	
53	4. **Output** is the packing slip (what you produced and its specs).
54	
55	With these four blocks you can describe almost any AWS infrastructure.


## Note 55: HCL’s Four Building Blocks (Resource, Variable, Local, Output)

- id: `8e53af7c-88fb-49e9-b153-a4b0ab7bf852`
- last_edited: 2026-10-23T02:01:41.054Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "HCL’s Four Building Blocks (Resource, Variable, Local, Output)" [8e53af7c-88fb-49e9-b153-a4b0ab7bf852] 15 lines
--- Databases ---
none
1	# HCL’s Four Building Blocks <hepta-color type="text" color="yellow">(Resource, Variable, Local, Output)</hepta-color>
2	
3	If you automated infrastructure with Shell or AWS CLI, you know the pain: compute order exactly—“create the VPC, take the ID, then the Subnet, and clean up on failure.” That is **imperative** thinking, a long recipe.
4	
5	<hepta-color type="text" color="yellow">In Terraform we use </hepta-color><hepta-color type="text" color="yellow">**HCL（HashiCorp Configuration Language）**</hepta-color><hepta-color type="text" color="yellow">. It is not a scripting language; it is a </hepta-color><hepta-color type="text" color="yellow">**declarative**</hepta-color><hepta-color type="text" color="yellow"> config language. You do not tell Terraform how; you describe the desired world with four blocks—</hepta-color><hepta-color type="text" color="yellow">**resource**</hepta-color><hepta-color type="text" color="yellow">、</hepta-color><hepta-color type="text" color="yellow">**variable**</hepta-color><hepta-color type="text" color="yellow">、</hepta-color><hepta-color type="text" color="yellow">**locals**</hepta-color><hepta-color type="text" color="yellow">、</hepta-color><hepta-color type="text" color="yellow">**output**</hepta-color><hepta-color type="text" color="yellow">. Terraform handles the rest.</hepta-color>
6	
7	<hepta-mention type="card" id="8fa2b62b-e31d-449c-b4e4-b360261d28e3">Resource: the atomic unit of infrastructure</hepta-mention>
8	
9	<hepta-mention type="card" id="01bdf71b-9a79-4cf0-9497-602fa63f8b84">Variable: flexibility and type safety</hepta-mention>
10	
11	<hepta-mention type="card" id="298762e1-5bf9-4b53-85ca-ae0d2762a827">Locals: internal computed constants and logic wrapping</hepta-mention>
12	
13	<hepta-mention type="card" id="57f964bb-6fcc-4476-a822-06f9e4663e71">Output: the public interface for sharing across modules</hepta-mention>
14	
15	<hepta-mention type="card" id="d4e8e090-25fa-4fd0-a2f3-870290bdc8c4">Implicit dependency: how Terraform orders work</hepta-mention>


## Note 56: Output: the public interface for sharing across modules

- id: `57f964bb-6fcc-4476-a822-06f9e4663e71`
- last_edited: 2026-10-23T01:59:38.810Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Output: the public interface for sharing across modules" [57f964bb-6fcc-4476-a822-06f9e4663e71] 20 lines
--- Databases ---
none
1	# Output: the public interface for sharing across modules
2	
3	The last block is `output`. If `variable` is input, `output` is **output (return values)**。
4	
5	Why output? Separation of duties. Later we put VPC in a Foundation Stack and the EKS cluster in a Service Stack.
6	
7	When the network layer finishes it must tell the world: what is the VPC ID? Which Private Subnet IDs?
8	
9	### As the Stack’s API
10	
11	`output` is the API this Stack exposes.
12	
13	```hcl
14	output "vpc_id" {
15	  description = "VPC ID created in this Stack"
16	  value       = aws_vpc.main.id
17	}
18	```
19	
20	<hepta-color type="text" color="yellow">**After apply, Terraform stores these outputs in State. Other Terraform projects can fetch them via **</hepta-color><hepta-color type="text" color="yellow">**`terraform_remote_state`**</hepta-color><hepta-color type="text" color="yellow">** like an API call—no copy-paste.**</hepta-color>


## Note 57: Locals: internal computed constants and logic wrapping

- id: `298762e1-5bf9-4b53-85ca-ae0d2762a827`
- last_edited: 2026-10-23T01:59:35.240Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Locals: internal computed constants and logic wrapping" [298762e1-5bf9-4b53-85ca-ae0d2762a827] 26 lines
--- Databases ---
none
1	# Locals: internal computed constants and logic wrapping
2	
3	Beginners ask: how do `locals` and `variable` differ?
4	
5	Good question. If `variable` is **passed in from outside**, then `locals` is the result of **internal computation**.
6	
7	In production, naming is often `project-env-region-function`. Assembling that string in every `resource` `tags` bloats the code and invites mistakes.
8	
9	### Mental model for locals: define once, use everywhere
10	
11	Wrap complex logic in `locals`:
12	
13	```hcl
14	locals {
15	  # Combine external variables into a shared prefix
16	  name_prefix = "${var.project}-${var.env}-${var.region}"
17	  
18	  # Constants that outsiders should not change, but many internals reuse
19	  common_tags = {
20	    Project   = var.project
21	    ManagedBy = "Terraform"
22	  }
23	}
24	```
25	
26	In `resource` reference with `local.<name>`. Change naming in one `locals` and every related resource updates. Maintainability goes up.


## Note 58: Variable: flexibility and type safety

- id: `01bdf71b-9a79-4cf0-9497-602fa63f8b84`
- last_edited: 2026-10-23T01:58:05.324Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Variable: flexibility and type safety" [01bdf71b-9a79-4cf0-9497-602fa63f8b84] 40 lines
--- Databases ---
none
1	# Variable: flexibility and type safety
2	
3	Hard-code CIDR or env name in `resource` and the code cannot be reused. Want both `dev` and `prod`? Copy two files? That is when `variable` appears.
4	
5	<hepta-color type="text" color="yellow">`variable`</hepta-color><hepta-color type="text" color="yellow"> is like a function’s </hepta-color><hepta-color type="text" color="yellow">**input parameters**</hepta-color><hepta-color type="text" color="yellow">. The infra description becomes a template; different inputs yield different results.</hepta-color>
6	
7	### A strong type system
8	
9	Terraform variables are not string substitution. The type system filters bad input before deploy.
10	
11	- **Primitive types**：`string` (string), `number` (number), `bool` (boolean).
12	
13	- **Collection types**：`list` (ordered list), `map` (key-value).
14	
15	- **Structured types**：`object` (object), with every field and its type spelled out.
16	
17	Example: an EKS node group config variable:
18	
19	```hcl
20	variable "node_config" {
21	  description = "Detailed config for the EKS node group"
22	  type = object({
23	    instance_type = string
24	    min_size      = number
25	    max_size      = number
26	  })
27	}
28	```
29	
30	With that type, if someone passes a string as `min_size`, Terraform fails at `plan` instead of failing in the cloud on a bad parameter.
31	
32	### How to reference
33	
34	In `resource` you get values with `var.<variable_name>`:
35	
36	```hcl
37	resource "aws_vpc" "main" {
38	  cidr_block = var.vpc_cidr  # reference the variable named vpc_cidr
39	}
40	```


## Note 59: Resource: the atomic unit of infrastructure

- id: `8fa2b62b-e31d-449c-b4e4-b360261d28e3`
- last_edited: 2026-10-23T01:57:57.614Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "Resource: the atomic unit of infrastructure" [8fa2b62b-e31d-449c-b4e4-b360261d28e3] 29 lines
--- Databases ---
none
1	# Resource: the atomic unit of infrastructure
2	
3	<hepta-color type="text" color="yellow">In Terraform, </hepta-color><hepta-color type="text" color="yellow">`resource`</hepta-color><hepta-color type="text" color="yellow"> is the block you see most. It is one concrete cloud thing: a server, a database, a network.</hepta-color>
4	
5	### Syntax breakdown
6	
7	A standard AWS VPC declaration:
8	
9	```hcl
10	resource "aws_vpc" "main" {
11	  cidr_block = "10.0.0.0/16"
12	
13	  tags = {
14	    Name = "primary-vpc"
15	  }
16	}
17	```
18	
19	Three core parts:
20	
21	1. **Provider type (****`aws_vpc`****)**: what to create. Prefix `aws` means an AWS Provider resource; `vpc` is the type inside that provider.
22	
23	2. **Local name (****`"main"`****)**: the nickname inside this Terraform code. It is **not** the cloud resource name (that comes from `tags` inside `Name` or other attributes). Other resources in this file use the nickname to refer to it.
24	
25	3. **Attribute block (****`{ ... }`****)**: inside the braces, the full spec. For example `cidr_block` sets the network range.
26	
27	### Why two names?
28	
29	Beginners mix up the `aws_vpc` after `"main"`. Remember: **the cloud unique ID (vpc-123abc...) exists only after apply**. While writing code <hepta-color type="text" color="yellow">you need a stable label—that is the local name.</hepta-color>


## Note 60: The IaC Mindset Shift: From ClickOps to Declarative Ops

- id: `1c306c42-5b39-4e51-ba50-2a9b26bb32df`
- last_edited: 2026-10-23T01:55:34.656Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "The IaC Mindset Shift: From ClickOps to Declarative Ops" [1c306c42-5b39-4e51-ba50-2a9b26bb32df] 118 lines
--- Databases ---
none
1	# <hepta-color type="text" color="yellow">The IaC Mindset Shift: From ClickOps to Declarative Ops</hepta-color>
2	
3	Have you spent half an hour in the AWS Console building a VPC and EKS cluster, then been told to clone it in another region? Worse: prod dies, and a colleague tweaked a Security Group a month ago "just for a moment" — nobody remembers why.
4	
5	If infra living only in someone's head or notebook makes you anxious, this lesson's **Infrastructure as Code (IaC)** mindset shift exists to end that chaos.
6	
7	## Why ClickOps is a dead end in production
8	
9	Early on, clicking the Web Console (**ClickOps**) is fast and intuitive. Fine for experiments. In production ops, ClickOps has five fatal problems:
10	
11	### <hepta-color type="text" color="yellow">1. Non-repeatability</hepta-color>
12	
13	A second or third environment (int → prod) depends on memory, screenshots, or a Notion SOP. Miss one checkbox and the stacks diverge. That is why bugs appear only in prod.
14	
15	### <hepta-color type="text" color="yellow">2. Lack of traceability</hepta-color>
16	
17	Who changed the DB instance type? Who opened 0.0.0.0/0 SSH for debug and forgot to close it? Without CloudTrail expertise, change history is broken. You cannot read infra evolution like Git commits.
18	
19	### <hepta-color type="text" color="yellow">3. Configuration drift</hepta-color>
20	
21	The DevOps nightmare. People make "small" Console edits that live nowhere. Cloud reality slowly detaches from what you think (or the docs). A large update then hits hidden landmines.
22	
23	### <hepta-color type="text" color="yellow">4. Implicit dependencies</hepta-color>
24	
25	The Console hides work (e.g. a default Security Group). Convenient, until you delete something five unknown resources depend on and teardown becomes a disaster.
26	
27	### <hepta-color type="text" color="yellow">5. Hard to review</hepta-color>
28	
29	Nobody can review before you click. Errors happen after the fact. IaC produces a Plan first so the team can check safety and architecture.
30	
31	## Declarative thinking: this is how the world should look
32	
33	<hepta-color type="text" color="yellow">Entering Terraform requires the first, most important shift: from </hepta-color><hepta-color type="text" color="yellow">**imperative**</hepta-color><hepta-color type="text" color="yellow"> to </hepta-color><hepta-color type="text" color="yellow">**declarative**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
34	
35	Imagine hiring someone to build a house:
36	
37	- **Imperative**:
38	  "Buy 500 bricks, dig 2 meters at A, pour, stack..."
39	  You specify every **step**. Miss one or get order wrong and the house fails. Ansible and shell scripts are usually this.
40	
41	- <hepta-color type="text" color="yellow">**Declarative**</hepta-color><hepta-color type="text" color="yellow">:
42	  "I want two floors, three rooms, red walls."
43	  You define </hepta-color><hepta-color type="text" color="yellow">**desired state**</hepta-color><hepta-color type="text" color="yellow">. How to buy bricks and dig is Terraform's job.</hepta-color>
44	
45	### Why declarative fits infrastructure
46	
47	Cloud resources have hard order: VPC before Subnet; Subnet before EKS Node Group.
48	
49	Imperative means you sequence that yourself. A mid-run failure needs messy "already half built" logic.
50	
51	Terraform is declarative. List every resource; <hepta-color type="text" color="yellow">Terraform builds a </hepta-color><hepta-color type="text" color="yellow">**dependency graph**</hepta-color><hepta-color type="text" color="yellow">. A depends on B, so B first. If a resource already matches, it does nothing; if not, it updates. Aligning to desired state is IaC's core.</hepta-color>
52	
53	| Trait | Imperative | <hepta-color type="text" color="yellow">Declarative</hepta-color> |
54	| --- | --- | --- |
55	| **Focus** | Process and steps | <hepta-color type="text" color="yellow">Desired end state</hepta-color> |
56	| **Examples** | Shell Script, Ansible | <hepta-color type="text" color="yellow">**Terraform**</hepta-color><hepta-color type="text" color="yellow">, CloudFormation</hepta-color> |
57	| **Pros** | Flexible control flow | <hepta-color type="text" color="yellow">Auto dependencies, idempotency</hepta-color> |
58	| **Cons** | Hard to track real current state | <hepta-color type="text" color="yellow">Must learn a description language (HCL)</hepta-color> |
59	
60	## Where Terraform sits on the IaC spectrum
61	
62	Many IaC tools exist. Why Terraform? Position:
63	
64	1. **Cloud agnostic**:
65	   Unlike CloudFormation (AWS only), Terraform **Provider**s manage AWS, GCP, Azure, plus Kubernetes, Cloudflare DNS, Datadog, and more. One language for the stack.
66	
67	2. <hepta-color type="text" color="yellow">**HCL (HashiCorp Configuration Language)**</hepta-color><hepta-color type="text" color="yellow">: Designed for infra. More readable than JSON (comments, no brace hell); stricter than YAML (types).  Human-readable vs machine-friendly.</hepta-color>
68	
69	3. <hepta-color type="text" color="yellow">**Provider ecosystem**</hepta-color><hepta-color type="text" color="yellow">:
70	   1000+ official and community providers. If it has an API, you can usually automate it with Terraform.</hepta-color>
71	
72	## State: Terraform's memory
73	
74	Last technical mindset shift this lesson.
75	
76	<hepta-color type="text" color="yellow">You write </hepta-color><hepta-color type="text" color="yellow">`resource "aws_vpc" "main" {}`</hepta-color><hepta-color type="text" color="yellow">, apply, and AWS gets a VPC </hepta-color><hepta-color type="text" color="yellow">`vpc-12345`</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
77	
78	<hepta-color type="text" color="yellow">Next day you change code to </hepta-color><hepta-color type="text" color="yellow">`cidr_block = "10.0.1.0/16"`</hepta-color><hepta-color type="text" color="yellow">. On apply, how does Terraform </hepta-color> <hepta-color type="text" color="yellow">know to update </hepta-color><hepta-color type="text" color="yellow">`vpc-12345`</hepta-color><hepta-color type="text" color="yellow"> instead of creating a new VPC?</hepta-color>
79	
80	The answer is **State**。
81	
82	### <hepta-color type="text" color="yellow">State bridges code and the real world</hepta-color>
83	
84	<hepta-color type="text" color="yellow">State is Terraform's memory: which code block maps to which cloud resource (ID, current parameters).</hepta-color>
85	
86	<hepta-color type="text" color="yellow">Without State, Terraform is an architect with amnesia. Foundation already poured, but today's blueprint makes it try to pour again — then error because it exists.</hepta-color>
87	
88	### Why State management is critical
89	
90	Solo, State may be a local `terraform.tfstate` file. In a team, <hepta-color type="text" color="yellow">State must live somewhere safe and shared (e.g. S3), called </hepta-color><hepta-color type="text" color="yellow">**Remote Backend**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
91	
92	Good State management gives three wins:
93	
94	1. **Single source of truth**: the team shares one memory — no "I built a VPC you never heard of."
95	
96	2. **Locking**: while you update, Terraform locks State so a colleague cannot collide.
97	
98	3. **Drift detection**: comparing code, State, and the live cloud, Terraform tells you if someone edited the Console behind your back.
99	
100	## Summary: from concrete to abstract
101	
102	ClickOps to IaC is leaving concrete clicks for abstract description. You do not press buttons; you define the world.
103	
104	Remember: every HCL line is not a command. It is a **desired vision**. Terraform uses memory (State) and cloud APIs (providers) to make that vision real on AWS.
105	
106	## Learning foundation: lock the mental model
107	
108	Before we write code, internalize these. If they already feel obvious, you are ready:
109	
110	- **IaC solves drift and non-repeatability.**
111	
112	- **Declarative: you define the result; the tool chooses steps.**
113	
114	- **State is Terraform's only memory joining code and real resources.**
115	
116	- **Every change: Plan (preview) then Apply (mutate).**
117	
118	Next we unpack HCL and build your first AWS network from four basic blocks.


## Note 61: A production cloud platform is not one-shot magic. It stacks bottom-up. Each layer's output is the next layer's input.

- id: `91aa38c1-1c2c-431e-a86c-f504761911dd`
- last_edited: 2026-11-07T02:25:22.368Z
- whiteboards: DevOp All, AWS Infrastructure with Terraform

card "A production cloud platform is not one-shot magic. It stacks bottom-up. Each layer's output is the next layer's input." [91aa38c1-1c2c-431e-a86c-f504761911dd] 61 lines
--- Databases ---
none
1	# <hepta-color type="text" color="yellow">A production cloud platform is not one-shot magic.</hepta-color> <hepta-color type="text" color="yellow">It stacks bottom-up. Each layer's output is the next layer's input.</hepta-color> 
2	
3	We build your AWS platform in six phases:
4	
5	![](heptabase://card/91aa38c1-1c2c-431e-a86c-f504761911dd/file/98b8ae58-6e3a-4166-b7a0-bf2184dd1a23)
6	
7	> *Cloud platform build order: each layer sits on a solid foundation below.*
8	
9	### Phase 1: Network foundation
10	
11	Start of all cloud resources. We use Terraform to create **VPC**、**Subnet**、**NAT Gateway** and other network pieces, plus DNS (Route53), certs (ACM), and base Security Groups. Without this boundary, services have nowhere to live.
12	
13	### Phase 2: Compute cluster (the platform cluster)
14	
15	Network first, then compute. We create **EKS Cluster** and **Node Groups**. More important: **OIDC Provider** and **IRSA Roles**, so Kubernetes Pods can assume AWS IAM Roles and reach AWS APIs.
16	
17	### Phase 3: Cluster capabilities
18	
19	Bare EKS is a shell. We install AWS Load Balancer Controller (ALBs), External DNS (domain sync), and Cluster Autoscaler (node count). Those hook K8s to native cloud services.
20	
21	### Phase 4: Observability
22	
23	Production cannot be a black box. We deploy Prometheus, Grafana, Loki, and Tempo for metrics, logs, and traces — before business apps go live.
24	
25	### Phase 5: Business workloads
26	
27	Your APIs, data pipelines, ML models. We handle Secrets and deploy with Helm.
28	
29	### Phase 6: Automation (safe delivery)
30	
31	Finally automate. **GitHub Actions** CI/CD pipelines: reviewed changes, plus drift detection when cloud reality diverges from code.
32	
33	## Two repositories, split duties
34	
35	In practice, keep infrastructure code and application code apart — cleanliness, blast radius, and permission isolation.
36	
37	This course centers on two repos:
38	
39	### 1. infra-repo (Terraform)
40	
41	Its job is **managing AWS resources**。
42	
43	- **Input:** environment var files (`.tfvars`）。
44	
45	- **Output:** resource IDs (VPC ID, Subnet ID, EKS cluster name, IAM Role ARN, etc.).
46	
47	- Think site prep and utilities.
48	
49	### 2. k8s-app-repo (Helmfile)
50	
51	Its job is **everything on Kubernetes**。
52	
53	- **Input:** the outputs from `infra-repo`.
54	
55	- **Task:** install Helm charts from those outputs, set Ingress cert ARNs, pick which IAM Role a Pod uses.
56	
57	- Think building and furnishing on that land.
58	
59	The join matters. Terraform creates EKS and an IAM Role ARN; you put that value in Helmfile so K8s objects know how to attach to cloud identity.
60	
61


## Note 62: Release Versioning

- id: `e78648f4-34f4-44c6-af3f-c651559d5de7`
- last_edited: 2026-10-13T13:20:40.399Z
- whiteboards: DevOp All, Helm and Helmfile

card "Release Versioning" [e78648f4-34f4-44c6-af3f-c651559d5de7] 104 lines
--- Databases ---
none
1	# Release Versioning
2	
3	Last part we discussed why, in enterprise practice, we prefer `install` over tearing everything down and starting over (`upgrade`). That is not only about less downtime.<hepta-color type="text" color="yellow">The deeper reason:</hepta-color><hepta-color type="text" color="yellow">`upgrade`</hepta-color><hepta-color type="text" color="yellow"> triggers Helm's versioning so we always have an undo button ready.</hepta-color>
4	
5	Imagine Friday at 5 p.m. you ship a seemingly perfect update, then monitoring explodes. Without Helm you scramble for the last YAML, confirm the Image Tag, and run `kubectl apply` by hand. With Helm it is one command in seconds. How does this version control work inside the cluster? Let's open it up.
6	
7	## The cluster-state brain: where is Release data stored?
8	
9	When you run `helm install` or `helm upgrade`, Helm does more than send resources to the API Server—it also takes notes in your cluster.
10	
11	### Helm 3's decentralized design
12	
13	In Helm 2 the cluster needed Tiller (server-side) talking to the local Helm CLI over gRPC. Tiller stored state in its own DB or ConfigMaps, which brought serious security and architectural complexity.
14	
15	<hepta-color type="text" color="yellow">Helm 3 dropped that design for a Tiller-less architecture. Helm is now a pure client that stores its own state in native Kubernetes resources. Concretely, Helm stores all Release info as Kubernetes </hepta-color><hepta-color type="text" color="yellow">**Secret**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
16	
17	### Why Secrets?
18	
19	<hepta-color type="text" color="yellow">The choice is clever, with several benefits:</hepta-color>
20	
21	1. <hepta-color type="text" color="yellow">**No external database**</hepta-color><hepta-color type="text" color="yellow">: Kubernetes itself is Helm's database. As long as the cluster lives, deploy history is not lost.</hepta-color>
22	
23	2. <hepta-color type="text" color="yellow">**Security (RBAC)**</hepta-color><hepta-color type="text" color="yellow">: because the data lives in Secrets, you can use standard Kubernetes RBAC to control who reads or changes Release history. If you cannot read Secrets in a Namespace, you cannot use </hepta-color><hepta-color type="text" color="yellow">`helm list`</hepta-color><hepta-color type="text" color="yellow"> to see Releases there.</hepta-color>
24	
25	3. <hepta-color type="text" color="yellow">**Namespace isolation**</hepta-color><hepta-color type="text" color="yellow">: Release data is stored in the same Namespace as the Release. That matches Kubernetes multi-tenant logic.</hepta-color>
26	
27	If you are curious what this looks like, try:
28	
29	```bash
30	kubectl get secrets -n <your-namespace> | grep sh.helm.release
31	```
32	
33	You will see resources like `sh.helm.release.v1.my-app.v1`、`sh.helm.release.v1.my-app.v2`. Each Secret is one Release revision, with the full rendered YAML, original Values, and metadata.
34	
35	## Reading `helm history`: a digital trail on the timeline
36	
37	To see how an app evolved, the most direct command is `helm history <release-name>`. That output is an ops engineer's first stop when debugging.
38	
39	### Key fields
40	
41	A typical `history` output looks like this:
42	
43	| REVISION | UPDATED | STATUS | CHART | APP VERSION | DESCRIPTION |
44	| --- | --- | --- | --- | --- | --- |
45	| 1 | Mon Oct 23 10:00:00 2023 | superseded | my-app-1.0.0 | 3.0.0 | Install complete |
46	| 2 | Mon Oct 23 11:00:00 2023 | superseded | my-app-1.1.0 | 3.1.0 | Upgrade complete |
47	| 3 | Mon Oct 23 12:00:00 2023 | deployed | my-app-1.1.0 | 3.1.0 | Upgrade complete |
48	
49	Two easy-to-mix concepts need a clear split:
50	
51	- **REVISION**: the serial number of this deploy, starting at 1 and **monotonically increasing**. Whether you change one env var in `values.yaml` or replace the whole app, one `upgrade` adds 1.
52	
53	- **CHART VERSION**: this comes from the `Chart.yaml` `version` field. It is the version of the recipe itself.
54	
55	  - *Case study*: if you deploy the same Chart version (for example v1.1.0) twice just to tweak a parameter, you will see **REVISION become 3, while CHART stays at v1.1.0**。
56	
57	### What STATUS means
58	
59	- **deployed**: the version currently running in the cluster. A Release has only one `deployed` status.
60	
61	- **superseded**: ran before, now replaced by a newer version.
62	
63	- **failed**: the deploy failed. Maybe a Hook failed, YAML was invalid (though `upgrade` checks first, some logic errors happen at the API Server), or it timed out.
64	
65	- **uninstalled**: if you keep history when deleting a Release, you see this status.
66	
67	## The truth about Rollback: a forward fix
68	
69	To roll back you run `helm rollback <release-name> <revision-number>`. It sounds like rewind, but the underlying logic is very DevOps.
70	
71	### History only grows
72	
73	<hepta-color type="text" color="yellow">Helm's design philosophy:</hepta-color><hepta-color type="text" color="yellow">**version history should be an immutable audit trail**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
74	
75	<hepta-color type="text" color="yellow">So Rollback does not delete and rewind to an old Revision. It </hepta-color><hepta-color type="text" color="yellow">**reads the target Revision and uses it as a blueprint for a brand-new Revision**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
76	
77	Look at this flow:
78	
79	```mermaid:preview
80	graph LR
81	    A[Rev 1: initial install] --> B[Rev 2: bad config]
82	    B --> C[Rev 3: attempted fix, still failed]
83	    C -- "helm rollback my-app 1" --> D[Rev 4: restored to initial state]
84	    
85	    style D fill:#f9f,stroke:#333,stroke-width:4px
86	```
87	
88	- **Initial state**: Revision 1 (healthy).
89	
90	- **Bad change**: ran `upgrade` and became Revision 2 (broken).
91	
92	- **Another try**: ran `upgrade` and became Revision 3 (still failing).
93	
94	- **Rollback**: order a rollback to 1, which produces **Revision 4**. Revision 4's content (templates and values) matches Revision 1 exactly.
95	
96	**Why this design?**
97	
98	1. **Traceability**: you can see who rolled back and when.
99	
100	2. **No version chaos**: if rollback deleted Revisions 2 and 3, team discussions would not line up on version numbers.
101	
102	3. **Consistency**: to Kubernetes this is an ordinary update whose content happens to match the past. Hooks and resource control still run normally.
103	
104


## Note 63: Go Context Package in a Nutshell

- id: `54c5a4f8-e9a9-44d4-960b-24cc5aff0d57`
- last_edited: 2026-10-12T02:47:25.560Z
- whiteboards: go-backend

card "Go Context Package in a Nutshell" [54c5a4f8-e9a9-44d4-960b-24cc5aff0d57] 206 lines
--- Databases ---
none
1	# Go Context Package in a Nutshell
2	
3	## 為什麼 Go 需要 `context`？它為了解決什麼痛點？
4	
5	在 Go 裡面，啟動一個背景任務非常容易，只要一行 `go doSomething()` 就建立了一個 Goroutine。
6	
7	但也因為太容易了，帶來了兩個嚴重的後端難題：
8	
9	### <hepta-color type="text" color="yellow">難題 1：Goroutine 洩漏（Goroutine Leak）與浪費資源</hepta-color>
10	
11	<hepta-color type="text" color="yellow">假設使用者發出一個請求：「請幫我產生過去一年的報表」。後端 Handler 啟動了一個 Goroutine，要花 10 秒去抓資料。</hepta-color>
12	
13	<hepta-color type="text" color="yellow">結果過了 1 秒，使用者覺得太慢，直接按了「上一頁」或把瀏覽器分頁關閉。</hepta-color>
14	
15	<hepta-color type="text" color="yellow">這時候，</hepta-color><hepta-color type="text" color="yellow">**那個正在跑 10 秒的 Goroutine 知道使用者已經離開了嗎？**</hepta-color>
16	
17	<hepta-color type="text" color="yellow">答案是：</hepta-color><hepta-color type="text" color="yellow">**它完全不知道！**</hepta-color><hepta-color type="text" color="yellow"> 如果沒有外部通知機制，它依然會繼續霸占 CPU 與記憶體跑完剩下的 9 秒，這就是所謂的資源浪費與 Goroutine 洩漏。</hepta-color>
18	
19	### <hepta-color type="text" color="yellow">難題 2：跨層級通知（Cancellation Tree）</hepta-color>
20	
21	在真實的後端架構中，呼叫鏈往往是一層接一層的：
22	
23	$$
24	\text{Handler} \longrightarrow \text{Service} \longrightarrow \text{Repository (SQL)} \longrightarrow \text{外部 API}
25	$$
26	
27	<hepta-color type="text" color="yellow">如果最上層想要設定「這個請求如果超過 2 秒沒回應就全部停下來」，你該怎麼同時通知底下的 Service、資料庫查詢與外部 API 全部立即終止？</hepta-color>
28	
29	<hepta-color type="text" color="yellow">總不能把終止通知的 Channel 手動傳進每一個函式的參數裡吧？</hepta-color>
30	
31	---
32	
33	## <hepta-color type="text" color="yellow">`context`</hepta-color><hepta-color type="text" color="yellow"> 的本質：一棵「可以傳遞訊號與資料的樹」</hepta-color>
34	
35	為了解決上述問題，Go 官方在 Go 1.7 正式引入了 `context.Context` 介面。
36	
37	你只要記住一句話：<hepta-color type="text" color="yellow">**Context 是一棵樹，父節點被取消，所有的子節點都會跟著被取消。**</hepta-color>
38	
39	```text
40	               context.Background() (根節點)
41	                        │
42	                  r.Context() (HTTP 請求連線 Context)
43	                        │
44	        ┌───────────────┴───────────────┐
45	        ▼                               ▼
46	  WithTimeout (2秒超時)         WithValue (使用者資訊)
47	        │
48	  資料庫查詢 / 外部 API
49	```
50	
51	---
52	
53	## `context.Context` 介面到底長什麼樣？
54	
55	雖然功能強大，但它的介面其實只定義了 4 個方法：
56	
57	```go
58	type Context interface {
59	    Deadline() (deadline time.Time, ok bool)
60	    Done() <-chan struct{}
61	    Err() error
62	    Value(key any) any
63	}
64	```
65	
66	這 4 個方法裡，**最重要的只有一個：****`Done()`**。
67	
68	- **`Done() <-chan struct{}`**：
69	
70	  - 它是一個<hepta-color type="text" color="yellow">唯讀的 Channel。</hepta-color>
71	
72	  - 平常時候，這個 Channel 是阻塞的（收不到東西）。
73	
74	  - <hepta-color type="text" color="yellow">**一旦 Context 被取消（不管是因為超時、或是手動呼叫 cancel、或是客戶端斷線），Go 就會把這個 Channel 關閉（close）**</hepta-color><hepta-color type="text" color="yellow">！</hepta-color>
75	
76	  - <hepta-color type="text" color="yellow">在 Go 裡面，關閉的 Channel 會立刻解除阻塞，任何在監聽它的地方都會瞬間被喚醒。</hepta-color>
77	
78	- **`Err() error`**：
79	
80	  - 告訴你為什麼會結束。通常有兩種原因：`context.Canceled`（人為手動取消）或 `context.DeadlineExceeded`（時間到了超時）。
81	
82	---
83	
84	### 四、如何使用 Context？（從根節點到衍生節點）
85	
86	#### 1. 建立根節點（Root）
87	
88	<hepta-color type="text" color="yellow">樹一定要有根。Go 提供了兩種起點：</hepta-color>
89	
90	- <hepta-color type="text" color="yellow">`context.Background()`</hepta-color><hepta-color type="text" color="yellow">：最標準的根節點。通常用在 </hepta-color><hepta-color type="text" color="yellow">`main()`</hepta-color><hepta-color type="text" color="yellow"> 函式、背景主迴圈或單元測試的開頭。</hepta-color>
91	
92	- `context.TODO()`：當你還不知道要用什麼 Context，暫時先擺一個佔位符號時使用。
93	
94	#### 2. 衍生出具有「控制能力」的子節點
95	
96	從根節點開始，你可以用以下三種方式長出子節點：
97	
98	##### A. 手動取消：`context.WithCancel(parent)`
99	
100	```go
101	// 產生一個新的子 context，以及一個專門用來取消它的函式 cancel()
102	ctx, cancel := context.WithCancel(context.Background())
103	
104	// 當業務邏輯執行完畢，或發生錯誤時呼叫 cancel()
105	defer cancel()
106	```
107	
108	##### B. 時間超時自動取消：`context.WithTimeout(parent, duration)`（後端最常用）
109	
110	```go
111	// 設定這個任務最多只能執行 2 秒，時間一到，Go 自動發出取消訊號
112	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
113	defer cancel() // 好習慣：即使時間沒到就提前跑完，也呼叫 cancel 釋放計時器資源
114	```
115	
116	##### C. 攜帶鍵值資料：`context.WithValue(parent, key, val)`
117	
118	```go
119	// 在 context 樹上掛載這次請求的專屬資料（例如登入的使用者 ID）
120	ctx := context.WithValue(parent, "user_id", 42)
121	```
122	
123	---
124	
125	### 五、極簡程式碼範例：工人如何聽從指揮官的撤退訊號？
126	
127	我們寫一個最直觀的例子：一個工人在背後做耗時工作，指揮官設定 2 秒超時：
128	
129	```go
130	package main
131	
132	import (
133	    "context"
134	    "fmt"
135	    "time"
136	)
137	
138	// 工人函式：接收 ctx
139	func worker(ctx context.Context) {
140	    fmt.Println("工人：開始執行耗時任務...")
141	
142	    for {
143	        select {
144	        case <-ctx.Done(): // 關鍵：監聽指揮官的撤退訊號！
145	            fmt.Println("工人：收到取消訊號，原因：", ctx.Err())
146	            fmt.Println("工人：清理資源，提早收工！")
147	            return
148	        default:
149	            // 模擬正常工作
150	            fmt.Println("工人：工作中...")
151	            time.Sleep(500 * time.Millisecond)
152	        }
153	    }
154	}
155	
156	func main() {
157	    // 指揮官：設定這個任務最多只能跑 2 秒
158	    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
159	    defer cancel()
160	
161	    // 啟動工人
162	    go worker(ctx)
163	
164	    // 主程式等待 3 秒觀察結果
165	    time.Sleep(3 * time.Second)
166	    fmt.Println("主程式：結束")
167	}
168	```
169	
170	執行結果會是：
171	
172	1. 工人工作了幾次（每 0.5 秒一次）。
173	
174	2. 到了第 2 秒，`WithTimeout` 自動關閉了 `ctx.Done()` 的 Channel。
175	
176	3. 工人的 `select` 立刻命中 `case <-ctx.Done():`，印出取消原因 `context deadline exceeded`，優雅退出，絕不浪費多餘資源！
177	
178	---
179	
180	### Context 在 HTTP Web Server 中是怎麼運作的？
181	
182	現在你已經有了 `context` 的完整心智模型，我們回頭看 Chapter 12，這一切就豁然開朗了：
183	
184	1. **誰是指揮官？**
185	
186	   在 Web 伺服器中，**Go 的 ****`net/http`**** 伺服器就是指揮官** 。
187	
188	   當客戶端連線進來，伺服器底層自動建立了一個 Context，綁定在該 TCP 連線上，並透過 `r.Context()` 提供給你 。
189	
190	2. **什麼時候指揮官會發出撤退訊號（關閉 ****`ctx.Done()`****）？**
191	
192	   - 使用者等不及，按停止或關閉瀏覽器分頁時（客戶端斷線） 。
193	
194	   - 伺服器設定的讀寫超時時間到了 。
195	
196	   - 你的 Handler 執行結束回傳時 。
197	
198	3. **後端工程師的職責是什麼？**
199	
200	   你的職責就是：**永遠把 ****`r.Context()`**** 往下傳遞給資料庫驅動（例如 pgx）、Redis 客戶端或外部 HTTP 請求** 。
201	
202	   這樣一來，只要使用者一離開，你的資料庫查詢就會因為收到 `ctx.Done()` 而立刻中斷，完全不會浪費伺服器一絲一毫的效能！
203	
204	4. **攜帶資訊（****`r.WithContext`****）**
205	
206	   當你想把 Auth 驗證出來的 User ID 傳給後面的 Handler 時，你不需要全域變數，只需要在樹上衍生出一個帶有 Value 的新節點，並用 `r.WithContext(newCtx)` 包裝回傳即可 。


## Note 64: .gotmpl Dynamic Templates

- id: `2fcfef45-cd47-4279-93ce-611c7adc69d5`
- last_edited: 2026-10-11T15:06:01.678Z
- whiteboards: DevOp All, Helm and Helmfile

card ".gotmpl Dynamic Templates" [2fcfef45-cd47-4279-93ce-611c7adc69d5] 189 lines
--- Databases ---
none
1	# .gotmpl Dynamic Templates
2	
3	Imagine a CI/CD pipeline. Every code update produces a new `IMAGE_TAG` (for example a git commit hash). With a purely static `values.yaml`, you might rewrite the file with `sed` or `yq` before deploy, or pass a long list of `--set image.tag=xxx`. That is hard to maintain and easy to get wrong. Miss one env var and the deploy still runs — the cluster runs the wrong version, or the service goes down. Can a config file fetch env vars itself, and refuse to run when a required value is missing? That is what `.gotmpl` files are for in Helmfile.
4	
5	## The extension trick: YAML vs .gotmpl
6	
7	In Helmfile the file extension is not decoration. It decides how Helmfile processes the file.
8	
9	When `helmfile.yaml` references a file in the `values` block, Helmfile branches on the suffix:
10	
11	- <hepta-color type="text" color="yellow">**`.yaml`**</hepta-color><hepta-color type="text" color="yellow">** files**</hepta-color><hepta-color type="text" color="yellow">: treated as static text. Helmfile reads them and hands them to Helm unchanged. If you write </hepta-color><hepta-color type="text" color="yellow">`.yaml`</hepta-color><hepta-color type="text" color="yellow"> inside a </hepta-color><hepta-color type="text" color="yellow">`{{ .Values.xxx }}`</hepta-color><hepta-color type="text" color="yellow">, it is not evaluated. It is passed into Kubernetes as a string, which usually breaks YAML.</hepta-color>
12	
13	- <hepta-color type="text" color="yellow">**`.gotmpl`**</hepta-color><hepta-color type="text" color="yellow">** files**</hepta-color><hepta-color type="text" color="yellow">: a Go template. Helmfile renders it first — variables, functions, logic — then gives the resulting YAML to Helm.</hepta-color>
14	
15	That turns a dead config file into a small program. You can generate the final config from the current environment, AWS region, or CI parameters.
16	
17	## requiredEnv: a safety net for deploys
18	
19	In production some values must never be missing, for example:
20	
21	1. **Image Tag**: so you deploy the right code version.
22	
23	2. **API Key / Secret**: so the service can talk to external resources.
24	
25	3. **Region / Cluster Name**: so resources land in the right place.
26	
27	A plain `env` function (for example `{{ env "TAG" }}`) returns an empty string if the env var is unset. That is dangerous: `image: my-app:` (empty tag) can make Kubernetes pull the `latest` image. That is not what you want.
28	
29	### Syntax and how errors work
30	
31	`requiredEnv` solves this. The syntax is straightforward:
32	
33	```yaml
34	# values.yaml.gotmpl
35	image:
36	  repository: my-repo/api
37	  tag: {{ requiredEnv "IMAGE_TAG" }}
38	```
39	
40	When you run `helmfile apply`, if the shell has no `IMAGE_TAG` defined, Helmfile errors immediately and stops. The message is clear: IMAGE\_TAG is required but not set.
41	
42	This **fail-fast** behavior matters in GitOps. Incomplete config never reaches the cluster.
43	
44	### A practical example
45	
46	Suppose the service talks to Stripe, and Dev vs Prod use different API keys. You can design it like this:
47	
48	```yaml
49	# values.yaml.gotmpl
50	config:
51	  stripeKey: {{ requiredEnv "STRIPE_API_KEY" }}
52	  environment: {{ .Environment.Name }}
53	  logLevel: {{ env "LOG_LEVEL" | default "info" }}
54	```
55	
56	In this example:
57	
58	- `STRIPE_API_KEY` is required. Missing it errors.
59	
60	- `LOG_LEVEL` is optional. If unset, `default` falls back to `"info"`。
61	
62	## The index function: navigating nested data
63	
64	Earlier we used Go template dot notation, for example `.Values.aws.region`. When the key itself is dynamic, dots do not work.
65	
66	### Why index?
67	
68	Imagine a lookup table of certificate ARNs by region:
69	
70	```yaml
71	# env/common.yaml
72	aws:
73	  certArns:
74	    us-east-1: "arn:aws:acm:us-east-1:123:cert/abc"
75	    us-west-2: "arn:aws:acm:us-west-2:123:cert/xyz"
76	    ap-northeast-1: "arn:aws:acm:ap-northeast-1:123:cert/def"
77	```
78	
79	<hepta-color type="text" color="yellow">At deploy time the current region is in </hepta-color><hepta-color type="text" color="yellow">`.Values.region`</hepta-color><hepta-color type="text" color="yellow">. You cannot write </hepta-color><hepta-color type="text" color="yellow">`{{ .Values.aws.certArns..Values.region }}`</hepta-color><hepta-color type="text" color="yellow"> — that is invalid. This is where </hepta-color><hepta-color type="text" color="yellow">`index`</hepta-color><hepta-color type="text" color="yellow"> earns its keep.</hepta-color>
80	
81	### <hepta-color type="text" color="yellow">Syntax</hepta-color>
82	
83	<hepta-color type="text" color="yellow">`index`</hepta-color><hepta-color type="text" color="yellow"> works like this: </hepta-color><hepta-color type="text" color="yellow">`index <Map object> <Key1> <Key2>...`</hepta-color><hepta-color type="text" color="yellow">. It walks the map with those keys. </hepta-color>
84	
85	<hepta-color type="text" color="yellow">In </hepta-color><hepta-color type="text" color="yellow">`.gotmpl`</hepta-color><hepta-color type="text" color="yellow"> you can write:</hepta-color>
86	
87	```yaml
88	# values.yaml.gotmpl
89	ingress:
90	  annotations:
91	    alb.ingress.kubernetes.io/certificate-arn: {{ index .Values.aws.certArns .Values.region }}
92	```
93	
94	When `.Values.region` is `"us-west-2"`, `index` looks up that key in the `certArns` map and fills in the matching `"arn:aws:acm:us-west-2:123:cert/xyz"`.  One template can look up the right resource ID from environment settings.
95	
96	## Static vs dynamic: a design choice
97	
98	Even though `.gotmpl` is powerful, not everything should be a template. Compare static YAML with `.gotmpl`:
99	
100	| Trait | Static YAML (`.yaml`) | Dynamic template (`.gotmpl`) |
101	| --- | --- | --- |
102	| **Readability** | Very high — you see the final result at a glance. | Lower — you must follow the logic to know the result. |
103	| **Flexibility** | Low — one file per environment. | High — one template for every environment. |
104	| **Data sources** | File contents only. | Env vars, environment config, built-in functions. |
105	| **Best for** | Settings that are the same everywhere (log format). | CI/CD variables, ARNs that change by region. |
106	
107	### Practice tips
108	
109	1. **Prefer injecting variables via ****`environments`**** to inject variables**: if replica counts differ by environment, define them in the `environments` block as `replicas`, then reference `{{ .Values.replicas }}` in the template, instead of writing complex `.gotmpl` inside `if-else`。
110	
111	2. **Use ****`requiredEnv`**** as a contract**: mark every CI/CD variable (image tag, etc.) as `requiredEnv` so you never deploy an empty tag.
112	
113	3. **Keep templates small**: if `.gotmpl` is full of logic, you over-designed. Push logic into environment-specific values files.
114	
115	## Worked example: a multi-region backend
116	
117	Combine this lesson: a deploy that switches by environment and requires an image tag.
118	
119	### Step 1: define environment variables
120	
121	In CI (GitHub Actions, etc.) you set:
122	
123	```bash
124	export IMAGE_TAG=v1.2.3
125	export DEPLOY_REGION=us-west-2
126	```
127	
128	### Step 2: design Environments in `helmfile.yaml` Environments
129	
130	Inject region into the `.Values` object:
131	
132	```yaml
133	# helmfile.yaml
134	environments:
135	  prod:
136	    values:
137	      - region: "us-west-2"
138	        certArns:
139	          us-west-2: "arn:aws:acm:us-west-2:..."
140	          us-east-1: "arn:aws:acm:us-east-1:..."
141	```
142	
143	### Step 3: write `values.yaml.gotmpl`
144	
145	This is the core dynamic config:
146	
147	```yaml
148	# values.yaml.gotmpl
149	# 1. Require the image tag from CI
150	image:
151	  tag: {{ requiredEnv "IMAGE_TAG" }}
152	
153	# 2. Pick the certificate from the current environment region
154	ingress:
155	  enabled: true
156	  certificateArn: {{ index .Values.certArns .Values.region }}
157	
158	# 3. Optional resource-limit overrides
159	resources:
160	  requests:
161	    cpu: {{ env "CPU_REQUEST" | default "100m" }}
162	    memory: {{ env "MEM_REQUEST" | default "256Mi" }}
163	```
164	
165	When this runs, Helmfile first checks that `IMAGE_TAG` exists, then pulls the ARN for `certArns` from `us-west-2`, then applies custom resource limits or defaults. The YAML Helm gets is clean and exact. That is the point of automated deploys.
166	
167	---
168	
169	## Logic layers and render order
170	
171	When you learn `.gotmpl`, remember this data-flow order. It is how you debug a missing variable:
172	
173	1. **Load environment values**: Helmfile first reads the values files in the `environments` block.
174	
175	2. **Build the ****`.Values`**** object**: merge those files into one global data pool.
176	
177	3. **Render ****`.gotmpl`**** files**: Helmfile starts the Go template engine and reads `.gotmpl`. The template can see two kinds of data:
178	
179	   - **External environment**: OS-level vars via `requiredEnv` or `env`.
180	
181	   - **Internal context**: the Step 2 merge, via `.Values`.
182	
183	4. **Hand off to Helm**: rendered plain YAML goes to the Helm CLI.
184	
185	With that order you can see why `.gotmpl` can mix a dynamic tag from CI (via `requiredEnv`) with static parameters from environment config (via `.Values`）。
186	
187	### What’s next
188	
189	You can inject static or layered values with `environments` and make them dynamic with `.gotmpl`. As apps grow, a single `helmfile.yaml` gets huge. Next: nested Helmfiles — splitting config into modules so you can compose a large cluster like building blocks.


## Note 65: Helmfile Architecture, Dataflow, and OIDC IAM Core Guide

- id: `817953d7-c6e1-4b7d-b31c-be28ed4b88ad`
- last_edited: 2026-10-11T14:58:58.828Z
- whiteboards: DevOp All, Helm and Helmfile

card "Helmfile Architecture, Dataflow, and OIDC IAM Core Guide" [817953d7-c6e1-4b7d-b31c-be28ed4b88ad] 164 lines
--- Databases ---
none
1	# Helmfile Architecture, Dataflow, and OIDC IAM Core Guide
2	
3	This card consolidates multi-environment orchestration in Helmfile, secret security (SOPS and External Secrets), reliable deployment (Atomic, Rolling Update, and PDB), and quality checks (Lint and Test) into a single architecture guide. It also explains OIDC IAM for cloud-native authentication.
4	
5	---
6	
7	## 1. Codebase Architecture (Directory Layout and Roles)
8	
9	A production-grade Helmfile repository uses a clean pattern: root entry, global base policy, environment layering, and domain-based child helmfiles:
10	
11	```text
12	helmfile-repository/
13	├── helmfile.yaml.gotmpl      # [Entry] Root entry file: only declares environments and includes child helmfiles
14	├── base.yaml                 # [Global Policy] Shared repositories and global helmDefaults (atomic, wait)
15	├── .sops.yaml                # [Key Mapping] Maps file paths to AWS KMS Key ARNs for SOPS
16	├── env/                      # [Environment Layer] Infrastructure constants (not tied to a single release)
17	│   ├── dev/
18	│   │   ├── global.yaml       # Shared dev variables (region, domain, cluster facts)
19	│   │   └── secrets.yaml      # SOPS-encrypted dev secrets
20	│   └── prod/
21	│       ├── global.yaml       # Shared prod variables
22	│       └── secrets.yaml      # SOPS-encrypted prod secrets
23	└── helmfiles/                # [Modular Orchestration] Domain child helmfiles to limit blast radius
24	    ├── kube-system/          # Cluster foundation (CSI Driver, Ingress Controller, Reloader)
25	    │   └── secrets-store-csi/
26	    │       ├── helmfile.yaml
27	    │       └── values.yaml.gotmpl
28	    ├── monitoring/           # Observability layer (Prometheus, Loki, Grafana)
29	    │   └── prometheus/
30	    │       ├── helmfile.yaml
31	    │       └── values.yaml.gotmpl
32	    └── services/             # Business microservices
33	        └── order-api/
34	            ├── helmfile.yaml # Inherits base.yaml, sets labels: { tier: backend, team: order }
35	            ├── values.yaml.gotmpl # Dynamic template reading env vars and CI parameters
36	            └── chart/        # Local chart with templates/, tests/, pdb.yaml, Chart.lock
37	```
38	
39	---
40	
41	## 2. Dataflow Architecture (Deployment Lifecycle)
42	
43	From opening a PR to CI/CD parameter injection, SOPS decryption, Helmfile template rendering, and Kubernetes cluster execution with cloud secrets:
44	
45	```text
46	+---------------------------------------------------------------------------------------------------------+
47	| [CI/CD Pipeline (GitHub Actions / GitLab CI)]                                                           |
48	|  - Generates short-lived inputs: IMAGE_TAG (commit hash), AWS_REGION                                    |
49	|  - Uses OIDC to assume an AWS IAM Role via AWS STS (No long-lived access keys needed)                    |
50	+----------------------------------------------------+----------------------------------------------------+
51	                                                     | Triggers: helmfile -e prod apply
52	                                                     v
53	+---------------------------------------------------------------------------------------------------------+
54	| [Helmfile Data Processing Engine]                                                                       |
55	|                                                                                                         |
56	| 1. Environment Layering                                                                                 |
57	|    Loads env/prod/global.yaml -> builds global {{ .Values }} (region, accountID, etc.)                  |
58	|                                                                                                         |
59	| 2. Just-In-Time Decryption (helm-secrets + SOPS + AWS KMS)                                              |
60	|    Reads env/prod/secrets.yaml -> SOPS calls AWS KMS (kms:Decrypt) -> decrypts in memory (never on disk)|
61	|                                                                                                         |
62	| 3. Dynamic Template Rendering (.gotmpl Rendering)                                                       |
63	|    Renders values.yaml.gotmpl                                                                           |
64	|      ├── requiredEnv "IMAGE_TAG" (Fail-fast: stops immediately if missing)                              |
65	|      ├── index .Values.certArns .Values.region (Dynamic map lookup)                                     |
66	|      └── Injects .Values from step 1 and decrypted secrets from step 2                                  |
67	|                                                                                                         |
68	| 4. Hands rendered plain YAML to Helm CLI                                                                |
69	+----------------------------------------------------+----------------------------------------------------+
70	                                                     | helm upgrade --install --atomic --wait --timeout 600
71	                                                     v
72	+---------------------------------------------------------------------------------------------------------+
73	| [Kubernetes Cluster Runtime]                                                                            |
74	|                                                                                                         |
75	| A. Safe Upgrade Protection                                                                              |
76	|    - RollingUpdate: maxSurge: 1, maxUnavailable: 0 (Zero downtime, no dropped requests)                 |
77	|    - PDB (PodDisruptionBudget): minAvailable: 2 (Protects against node drains and evictions)             |
78	|    - Atomic Watcher: Waits for Pods Ready; on timeout (600s) or failure, auto-rolls back to last revision|
79	|                                                                                                         |
80	| B. Runtime Secret Injection (External Secrets / Secrets Store CSI Driver)                               |
81	|    - Pod starts and mounts CSI Volume referencing SecretProviderClass                                   |
82	|    - CSI Driver calls AWS Provider                                                                      |
83	|    - AWS Provider uses Pod ServiceAccount token (IRSA / EKS OIDC) to fetch credentials from Secrets Mgr |
84	|    - Secrets mount directly into Pod tmpfs; never stored in plaintext Git                               |
85	|    - On secret rotation in AWS, Reloader detects changes and triggers a rolling restart                 |
86	|                                                                                                         |
87	| C. Dynamic QA Verification                                                                               |
88	|    - Runs helm test after deploy (spins up test Pods to verify network connectivity and endpoints)      |
89	+---------------------------------------------------------------------------------------------------------+
90	```
91	
92	---
93	
94	## 3. Consolidated Knowledge Summary
95	
96	1. **Declarative Orchestration and 4 Core Blocks**: Helmfile acts as Docker Compose for Helm. It organizes clusters through `environments` (env-specific values), `repositories` (chart sources), `helmDefaults` (global flags), and `releases` (applications to deploy).
97	
98	2. **Environment Layering and "Diff-Only"**: Common infrastructure variables live in `env/<env>/`, while application logic stays in `values.yaml.gotmpl`. All environments share the same template and inject only what differs, stopping copy-paste drift.
99	
100	3. **.gotmpl Dynamic Templates and Safety Guardrails**: `.gotmpl` enables Go template rendering. Use `requiredEnv` for fail-fast safety (stops the deploy if an image tag or secret is missing) and `index` for dynamic nested map lookups.
101	
102	4. **Modular Split and base.yaml**: Use `helmfiles:` to include child configs, keeping files readable and reducing blast radius. Children inherit repository lists and global policies via `bases: [base.yaml]`.
103	
104	5. **Label Selectors and GitOps Flow**: Tag releases with `labels` to target subsets using `-l tier=backend`. Follow standard GitOps: PR stage runs `template` (syntax check) and `diff` (visual change preview); merge stage runs `apply` and `test`.
105	
106	6. **Dual Version Pinning and Atomic Deploys**: Never use `latest` in production. Pin both Chart `version` (template logic) and `appVersion` (container image tag), and commit `Chart.lock`. Always deploy with three flags: `atomic: true` (rollback on failure), `wait: true` (wait for ready pods), and a clear `timeout: 600`.
107	
108	7. **Rolling Update and PDB Protection**: Set `maxSurge: 1, maxUnavailable: 0` to keep 100% capacity during upgrades. Pair this with `PodDisruptionBudget` (`minAvailable: 2`) so node drains (`kubectl drain`) never take down the whole service.
109	
110	8. **Encrypted Secrets in Git (SOPS + KMS)**: Plaintext secrets must never enter Git. SOPS uses envelope encryption to encrypt YAML values while keeping keys readable for code reviews. In CI/CD, `helm-secrets` decrypts in memory without writing secrets to disk.
111	
112	9. **Runtime Dynamic Secrets (External Secrets)**: Store database passwords directly in AWS Secrets Manager. Secrets Store CSI Driver mounts them into Pod memory via IRSA, and Reloader automates zero-downtime rolling restarts on secret rotation.
113	
114	10. **Quality Assurance Gates (QA)**: Static validation with `helm lint` catches syntax and SemVer errors early. Dynamic validation with `helm test` spins up test Pods to confirm real network endpoints and health after deployment.
115	
116	---
117	
118	## 4. What is OIDC IAM? (Deep Dive)
119	
120	### 1. What is IAM?
121	
122	**IAM (Identity and Access Management)** is the gatekeeper for cloud platforms like AWS.
123	
124	- **Identity**: Who is trying to connect? (A human user account, or a machine role like an IAM Role).
125	
126	- **Access (Policy)**: What are they allowed to do? (For example, read an S3 bucket, or call KMS decrypt via `kms:Decrypt`).
127	
128	**The traditional problem**: To let external workloads (like GitHub Actions runners or Kubernetes Pods) access AWS, engineers used to create long-lived `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` pairs. If someone accidentally committed these keys or lost them, attackers could compromise the entire AWS account.
129	
130	### 2. What is OIDC?
131	
132	**OIDC (OpenID Connect)** is an identity authentication layer built on top of OAuth 2.0.
133	
134	- **Simple analogy**: Think of "Sign in with Google" on a website. The website never sees your Google password. Google verifies your identity and issues a cryptographically signed badge (a JWT, JSON Web Token). The website verifies Google's signature and trusts who you are.
135	
136	### 3. Combining OIDC with IAM: Keyless Authentication
137	
138	**OIDC IAM** configures AWS IAM to directly trust external Identity Providers (such as GitHub Actions or an Amazon EKS cluster), trading short-lived tokens for AWS credentials:
139	
140	#### Scenario A: CI/CD Pipeline (GitHub Actions accessing AWS)
141	
142	1. During a CI run, GitHub issues a short-lived OIDC JWT stating: `I am the main branch of repo:my-org/my-repo`.
143	
144	2. The GitHub runner presents this JWT to AWS STS (Security Token Service) via `AssumeRoleWithWebIdentity`.
145	
146	3. AWS STS verifies GitHub's signature against an IAM Trust Policy (confirming the request comes from the exact repository and branch).
147	
148	4. Once verified, AWS issues **temporary credentials valid for 1 hour**.
149	
150	5. The runner uses these temporary credentials to call KMS for SOPS decryption or run `helmfile apply`.
151	
152	*Benefit: Zero long-lived AWS keys stored in your GitHub repository!*
153	
154	#### Scenario B: Inside Kubernetes (IRSA: IAM Roles for Service Accounts)
155	
156	1. The EKS cluster itself acts as an OIDC Identity Provider. When a Pod starts, Kubernetes mounts a projected ServiceAccount token into the Pod.
157	
158	2. The AWS Provider for Secrets Store CSI takes this token to AWS STS.
159	
160	3. AWS verifies that the token was signed by your EKS cluster and belongs to the authorized ServiceAccount in that namespace.
161	
162	4. AWS grants temporary IAM permissions to fetch secrets from AWS Secrets Manager.
163	
164	*Benefit: Pods need no hardcoded credentials. Permissions attach directly to Kubernetes ServiceAccounts, enforcing least privilege.*


## Note 66: Helmfile Role and Structure

- id: `591ebc6f-db39-4f90-b4c7-8a38f93493b9`
- last_edited: 2026-10-11T14:57:40.381Z
- whiteboards: DevOp All, Helm and Helmfile

card "Helmfile Role and Structure" [591ebc6f-db39-4f90-b4c7-8a38f93493b9] 159 lines
--- Databases ---
none
1	# Helmfile Role and Structure
2	
3	> <hepta-color type="text" color="yellow">Before advanced syntax, lock this in:</hepta-color>
4	>
5	> - <hepta-color type="text" color="yellow">**Helm**</hepta-color><hepta-color type="text" color="yellow"> packages and renders one Chart.</hepta-color>
6	>
7	> - <hepta-color type="text" color="yellow">**Helmfile**</hepta-color><hepta-color type="text" color="yellow"> coordinates and deploys many Charts.</hepta-color>
8	>
9	> <hepta-color type="text" color="yellow">On a microservice cluster Helmfile is not just a tool. It is a </hepta-color><hepta-color type="text" color="yellow">**deploy contract**</hepta-color><hepta-color type="text" color="yellow">. Config as files, env as variables, and a foundation for GitOps.</hepta-color>
10	
11	Picture production: Prometheus, Grafana, Nginx Ingress, cert-manager, plus five of your microservices. You need to update three services and bump Prometheus. You open a notebook: each Release's repo URL, version, `values.yaml` path, Namespace. One wrong `-f` and prod DB secrets land in staging. Past five Releases, a chain of Helm commands is an ops nightmare.
12	
13	## From manual commands to declarative orchestration
14	
15	Early Helm is `helm install` or `helm upgrade`. That is **imperative**: tell the computer "now, this Release, this action." But <hepta-color type="text" color="yellow">for large-scale infra we prefer </hepta-color><hepta-color type="text" color="yellow">**declarative**</hepta-color><hepta-color type="text" color="yellow">: describe the Releases you want and their states; let the tool converge.</hepta-color>
16	
17	That is **Helmfile**'s core value.
18	
19	### Role: Docker Compose for Helm
20	
21	With Docker, many containers means you stop running `docker run` and write a `docker-compose.yaml`. Helmfile is to Helm what Compose is to Docker.
22	
23	- **Docker Compose**: manage many containers.
24	
25	- **Helmfile**: manage many Helm Releases.
26	
27	Helmfile records Charts, Values, versions, and repos in one `helmfile.yaml`. The cluster is visible at a glance, and you can sync the same config to Dev, Staging, or Prod.
28	
29	### Why Helm CLI is not enough
30	
31	Two modes, many Releases:
32	
33	| Dimension | Helm CLI only | Helmfile |
34	| --- | --- | --- |
35	| **How you operate** | N times `helm upgrade` | One command `helmfile apply` |
36	| **Environments** | Manually switch `-f values-prod.yaml` | Via `-e prod` inject env vars |
37	| **Repo management** | Manual `helm repo add` | Declared in file, auto-updated at run |
38	| **Traceability** | Wiki or shell scripts | YAML in Git |
39	| **Error risk** | Wrong flags, wrong order (high) | File is the desired state (low) |
40	
41	## Four blocks of `helmfile.yaml` 
42	
43	A standard `helmfile.yaml` has four top-level blocks: where it runs (env), where Charts come from, how they run (global defaults), and what to run (the list).
44	
45	### 1. `environments`: environment switching
46	
47	<hepta-color type="text" color="yellow">This is Helmfile's strength. Define env tags (such as </hepta-color><hepta-color type="text" color="yellow">`int`</hepta-color><hepta-color type="text" color="yellow">, </hepta-color><hepta-color type="text" color="yellow">`stg`</hepta-color><hepta-color type="text" color="yellow">, </hepta-color><hepta-color type="text" color="yellow">`prod`</hepta-color><hepta-color type="text" color="yellow">) and a values file per environment.</hepta-color>
48	
49	```yaml
50	environments:
51	  int:
52	    values:
53	      - env/int/values.yaml
54	  prod:
55	    values:
56	      - env/prod/values.yaml
57	```
58	
59	When you run `helmfile -e prod apply`, Helmfile loads `env/prod/values.yaml`. One template, many environments, isolated config.
60	
61	### 2. `repositories`: Chart sources
62	
63	<hepta-color type="text" color="yellow">With Helm CLI we forget </hepta-color><hepta-color type="text" color="yellow">`helm repo add`</hepta-color><hepta-color type="text" color="yellow"> and Charts go missing. In Helmfile the repo list is in the file.</hepta-color>
64	
65	```yaml
66	repositories:
67	  - name: prometheus-community
68	    url: https://prometheus-community.github.io/helm-charts
69	  - name: bitnami
70	    url: https://charts.bitnami.com/bitnami
71	```
72	
73	Before any deploy, Helmfile checks and updates those repos. New teammates do not set up by hand — just `git clone`.
74	
75	### 3. `helmDefaults`: global defaults
76	
77	Avoid repetition. If every Release needs `atomic: true` (rollback on failed upgrade) or the same timeout, do not copy it into each Release.
78	
79	```yaml
80	helmDefaults:
81	  atomic: true     # rollback automatically on failed upgrade
82	  wait: true       # wait until Pods are Ready
83	  timeout: 600     # timeout
84	```
85	
86	### 4. `releases`: the deploy list
87	
88	The star of the file: which Releases to install. Each Release wraps a `helm upgrade --install` command.
89	
90	```yaml
91	releases:
92	  - name: prometheus
93	    namespace: monitoring
94	    chart: prometheus-community/kube-prometheus-stack
95	    version: 82.1.0
96	    values:
97	      - prometheus-values.yaml
98	```
99	
100	Name, Namespace, Chart path (or repo), pinned version (strongly recommended in prod), and matching `values.yaml`。
101	
102	## Full example: a complete `helmfile.yaml` 
103	
104	Put it together. A real monitoring stack (Prometheus and Loki):
105	
106	```yaml
107	# 1. Environments: which vars each env loads
108	environments:
109	  int:
110	    values:
111	      - env/int/global.yaml
112	  prod:
113	    values:
114	      - env/prod/global.yaml
115	
116	# 2. Repos: Charts must be findable
117	repositories:
118	  - name: prometheus-community
119	    url: https://prometheus-community.github.io/helm-charts
120	  - name: grafana
121	    url: https://grafana.github.io/helm-charts
122	
123	# 3. Global defaults: safer deploys
124	helmDefaults:
125	  atomic: true
126	  wait: true
127	  timeout: 600
128	
129	---
130	# 4. Release list: desired cluster state
131	releases:
132	  - name: prometheus
133	    namespace: monitoring
134	    chart: prometheus-community/kube-prometheus-stack
135	    version: 82.1.0
136	    values:
137	      - values/prometheus-values.yaml.gotmpl # dynamic templates allowed
138	
139	  - name: loki
140	    namespace: monitoring
141	    chart: grafana/loki
142	    version: 6.41.1
143	    values:
144	      - values/loki-values.yaml
145	```
146	
147	### Why this beats running Helm by hand
148	
149	1. **Self-documenting**：`helmfile.yaml` is the cluster architecture diagram.
150	
151	2. **Atomic operations**: with `helmDefaults`, each package update is safe. Half-failed Pods do not leave a wrecked cluster.
152	
153	3. **Version pinning**: you specify `82.1.0`. A bare `helm install` might grab latest and drift across environments.
154	
155	## Toward multi-environment management
156	
157	After the basic structure you may ask: if Dev and Prod differ only a little (DB host), do I need two fully different `releases` blocks?
158	
159	That is Helmfile **environment layering**. Switch environments and let one `releases` declaration inject different Values per env tag. Next we go deeper on the `environments` block — one template, every environment.


## Note 67: External Secrets Architecture

- id: `345db308-0e87-4ca1-89dc-dd98220ca5bb`
- last_edited: 2026-10-11T14:54:44.325Z
- whiteboards: DevOp All, Helm and Helmfile

card "External Secrets Architecture" [345db308-0e87-4ca1-89dc-dd98220ca5bb] 108 lines
--- Databases ---
none
1	# External Secrets Architecture
2	
3	Imagine the database password must change every 30 days. With SOPS you edit a file, re-encrypt, commit, and run CI/CD.<hepta-color type="text" color="yellow">If you change a number in the AWS console and the in-cluster app reads the new password in minutes, that is more elegant. That is today's cloud-native secrets architecture.</hepta-color>
4	
5	## From Git encryption to cloud hosting: SOPS vs Secrets Manager
6	
7	Last section we stored SOPS ciphertext in Git. That is convenient for config-layer secrets (env names, what to sync into Kubernetes).<hepta-color type="text" color="yellow">For runtime secrets (database passwords, third-party API keys), storing them in AWS </hepta-color><hepta-color type="text" color="yellow">**Secrets Manager**</hepta-color><hepta-color type="text" color="yellow"> is stronger.</hepta-color>
8	
9	Typical split of duties:
10	
11	- **SOPS (Git)**: manage Helmfile values. In `secrets.yaml` we define which AWS secrets to sync, not the actual passwords.
12	
13	- **AWS Secrets Manager**: store the real passwords. It <hepta-color type="text" color="yellow">supports rotation</hepta-color> and finer IAM control.
14	
15	## Secrets Store CSI (**Container Storage Interface)** Driver: architecture and split of work
16	
17	For a Kubernetes Pod to read an AWS secret you need an adapter of two parts: **Secrets Store CSI Driver** and **AWS Provider**。
18	
19	### Two-layer split
20	
21	Classic framework-plus-implementer design.
22	
23	1. **CSI Driver (framework)**：
24	
25	   - Installed as a DaemonSet on every node.
26	
27	   - Handles standard Kubernetes work such as mounting a volume onto a Pod.
28	
29	   - It does not know AWS. It is a generic secrets interface.
30	
31	2. **AWS Provider (implementer)**：
32	
33	   - Also a DaemonSet.
34	
35	   - <hepta-color type="text" color="yellow">The worker that calls Secrets Manager via the AWS SDK.</hepta-color>
36	
37	   - <hepta-color type="text" color="yellow">It needs </hepta-color><hepta-color type="text" color="yellow">**IRSA (IAM Roles for Service Accounts)**</hepta-color><hepta-color type="text" color="yellow"> to fetch values legally.</hepta-color>
38	
39	## Define the mapping: SecretProviderClass
40	
41	With the driver you still tell Kubernetes: **which AWS secret maps to which Kubernetes Secret or file path**. That is the `SecretProviderClass` CRD.
42	
43	A typical example:
44	
45	```yaml
46	apiVersion: secrets-store.csi.x-k8s.io/v1
47	kind: SecretProviderClass
48	metadata:
49	  name: my-app-secrets
50	spec:
51	  provider: aws                          # use the AWS provider
52	  parameters:
53	    objects: |                           # objects to fetch from AWS
54	      - objectName: "prod-db-password"   # name in Secrets Manager
55	        objectType: "secretsmanager"
56	  secretObjects:                         # sync into a K8s Secret (for envFrom)
57	    - secretName: my-app-k8s-secret
58	      type: Opaque
59	      data:
60	        - objectName: prod-db-password
61	          key: DB_PASSWORD               # env var name the Pod sees
62	```
63	
64	## Runtime flow: five inject steps
65	
66	When a Pod asks to mount these secrets, the mechanism is strict. From start to data:[\[6](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/9faa31d8-85b7-4be0-9aae-e1d3c6de4575?_agentLine=361-383#a6a8f4ef-0cf7-49cd-b575-a410fc5b93dd)
67	
68	![](heptabase://card/345db308-0e87-4ca1-89dc-dd98220ca5bb/file/f610e108-6e57-4ccb-9082-d4695fb3561f)
69	
70	> *Runtime inject flow: CSI Driver and AWS Provider authenticate across the cloud and move data*
71	
72	1. **Pod starts**: the scheduler sees a csi volume pointing at a given `SecretProviderClass`。
73	
74	2. **CSI Driver steps in**: the node CSI Driver sees the mount request and prepares a virtual disk.
75	
76	3. **Authenticate and call**: CSI Driver calls the AWS Provider. The provider uses the Pod's **ServiceAccount (IRSA)** to get temporary AWS credentials.
77	
78	4. **Fetch the secret**: with those credentials it calls Secrets Manager and gets the real password.
79	
80	5. **Mount and sync**: CSI Driver writes the value into a temp file in the Pod and, if configured, creates a matching `Secret` so the Pod can read env vars.
81	
82	## Auto-rotation: why Reloader?
83	
84	<hepta-color type="text" color="yellow">A common trap: </hepta-color><hepta-color type="text" color="yellow">**CSI Driver updates mounted files, not environment variables.**</hepta-color>
85	
86	<hepta-color type="text" color="yellow">When you change the password in the AWS console, CSI auto-rotation updates the file path in the Pod. Linux processes usually read env vars once at start and never again.</hepta-color>
87	
88	For true fully automatic updates we add **Reloader**：
89	
90	1. **Watch**: Reloader watches Kubernetes Secret objects.
91	
92	2. **Detect**: when CSI Driver finishes sync and updates the Secret, Reloader notices immediately.
93	
94	3. <hepta-color type="text" color="yellow">**Restart**</hepta-color><hepta-color type="text" color="yellow">: Reloader runs a </hepta-color><hepta-color type="text" color="yellow">**Rolling Restart**</hepta-color><hepta-color type="text" color="yellow"> on the related Deployment.</hepta-color>
95	
96	4. <hepta-color type="text" color="yellow">**Load**</hepta-color><hepta-color type="text" color="yellow">: new Pods repeat the five steps and read the latest env vars.</hepta-color>
97	
98	Together you get a cloud single source of truth: change AWS once, production rotates safely with no humans.
99	
100	### Building a stable production environment
101	
102	We now have the full puzzle from version pinning to secrets safety. Pinning Chart and app versions makes deploys predictable; `--atomic` and `--wait` make upgrades atomic; SOPS plus Secrets Store CSI Driver give flexible, deep defense. Next we add the last gate: `helm lint` and `helm test` for quality checks.
103	
104	---
105	
106	## Summary and what is next
107	
108	This section covered advanced External Secrets: centralize secrets in AWS Secrets Manager, inject them with CSI Driver and the AWS Provider, configure `SecretProviderClass`, and use Reloader with CSI auto-rotation for hands-free password updates. You can now build an enterprise-grade secure deploy flow. Next we validate these configs with automated tests.


## Note 68: Secrets Security Problems and SOPS

- id: `8902649c-73e1-4c7b-ab2e-5a4122d53bdf`
- last_edited: 2026-10-11T14:54:44.333Z
- whiteboards: DevOp All, Helm and Helmfile

card "Secrets Security Problems and SOPS" [8902649c-73e1-4c7b-ab2e-5a4122d53bdf] 202 lines
--- Databases ---
none
1	# Secrets Security Problems and SOPS
2	
3	## Start from a disaster: why your values.yaml smells like bait
4	
5	Imagine a teammate, for convenience, writes the production DB password into `values.yaml` and pushes it to GitHub or GitLab. They notice and delete the line at once, but the password is already forever in Git commit history. For an attacker it is bait in the dark: clone the repo and dig the plaintext out of history.
6	
7	DevOps wants everything as code, which creates a contradiction: we must version all config, but passwords, API keys, and TLS certs must never appear in Git as plaintext.
8	
9	That is today's core problem: how to manage secrets in Git safely without hurting team speed or CI/CD automation?
10	
11	## The original sin of plaintext secrets: not just Git
12	
13	Before the solution, be clear why plaintext secrets in a Helm Chart are extremely dangerous.
14	
15	### Git history is irreversible
16	
17	Git stores incrementally. Even if you delete a sensitive field in the latest version, it still lives in objects under `.git`. Unless you use a complex, destructive `git filter-repo` to rewrite history, that data stays with the repo forever.
18	
19	### Helm History exposure
20	
21	Helm itself is a risk too. Recall Release management: Helm 3 stores all Release info in a Kubernetes Secret.  When you run `helm get values <release-name>` or `helm history <release-name>`, Helm pulls those values from the cluster and prints them in the terminal.
22	
23	If you put a plaintext password in `values.yaml`, any engineer with read access to that Namespace can see every secret with one Helm command. That global transparency is never allowed in production.
24	
25	## AWS KMS: the invisible key guardian
26	
27	Encryption needs a root of trust. If you encrypt files with your own AES key, where do you store that key? If it also lives in Git, encryption is pointless. That is when **AWS KMS (Key Management Service)** enters.
28	
29	KMS philosophy:**Keys never leave the cloud; you can only ask me to use them.**
30	
31	### Why is KMS safe?
32	
33	1. **Permission bound to identity (IAM)**: you never download the key. You only need IAM permission (for example `kms:Decrypt`). If your laptop is stolen, an admin disabling your AWS account stops all further decrypts.
34	
35	2. **Hardware security module (HSM)**: KMS keys live in FIPS 140-2 Level 3 hardware; nobody can export the raw key.
36	
37	3. **Full audit**: every encrypt/decrypt request is logged in AWS CloudTrail, so you know who decrypted which file and when.
38	
39	### Environment isolation
40	
41	In production we usually give each environment (dev, staging, prod) its own KMS key (Master Key / Customer Master Key, CMK).
42	
43	- **Dev key**: open to all developers for test API keys.
44	
45	- **Prod key**: strictly limited to the CI/CD IAM Role and senior ops.
46	
47	Physical key separation means a stolen developer account cannot decrypt production files.
48	
49	## SOPS: encryption built for Git
50	
51	With KMS as the key center, we need a tool to encrypt files. That is Mozilla's **SOPS (Secrets OPerationS)**。
52	
53	SOPS's core feature is **partial encryption**。
54	
55	### Why partial encryption?
56	
57	Traditional tools (GPG, Ansible Vault) turn the whole file into noise. That is hostile to Git because:
58	
59	1. You cannot see file structure (key names) in code review.
60	
61	2. Changing one character makes Git Diff look like the whole file was replaced.
62	
63	<hepta-color type="text" color="yellow">SOPS is different: it </hepta-color><hepta-color type="text" color="yellow">**encrypts YAML values only, and leaves keys in plaintext**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
64	
65	A concrete comparison:
66	
67	```yaml
68	# Original secrets.yaml (plaintext)
69	db:
70	  user: admin
71	  password: my-super-secret-password # this needs encryption
72	  host: db.example.com
73	```
74	
75	After SOPS encrypts with KMS, the file becomes:
76	
77	```yaml
78	# SOPS-encrypted secrets.yaml
79	db:
80	  user: admin
81	  password: ENC[AES256_GCM,data:kf9d2h...,iv:...,tag:...] # only the value is encrypted
82	  host: db.example.com
83	sops: # encryption metadata lives here
84	  kms:
85	    - arn: arn:aws:kms:us-west-2:123456789012:key/abc-123...
86	  lastmodified: "2025-05-20T10:00:00Z"
87	  version: 3.9.0
88	```
89	
90	<hepta-color type="text" color="yellow">You can see </hepta-color><hepta-color type="text" color="yellow">`db.user`</hepta-color><hepta-color type="text" color="yellow"> and </hepta-color><hepta-color type="text" color="yellow">`db.host`</hepta-color><hepta-color type="text" color="yellow"> stay readable. Engineers can confirm which service and host the secret belongs to without decrypting.</hepta-color>
91	
92	### SOPS workflow
93	
94	<hepta-color type="text" color="yellow">SOPS uses </hepta-color><hepta-color type="text" color="yellow">**envelope encryption**</hepta-color><hepta-color type="text" color="yellow">:</hepta-color>
95	
96	1. <hepta-color type="text" color="yellow">SOPS generates a random symmetric Data Key locally.</hepta-color>
97	
98	2. <hepta-color type="text" color="yellow">The Data Key encrypts your file content.</hepta-color>
99	
100	3. <hepta-color type="text" color="yellow">SOPS calls KMS and encrypts the Data Key with the KMS Master Key.</hepta-color>
101	
102	4. <hepta-color type="text" color="yellow">The encrypted Data Key is stored with the ciphertext in the YAML </hepta-color><hepta-color type="text" color="yellow">`sops`</hepta-color><hepta-color type="text" color="yellow"> section.</hepta-color>
103	
104	On decrypt, SOPS sends the encrypted Data Key back to KMS, KMS returns the raw Data Key, and SOPS unlocks the file.
105	
106	## Hands-on: let .sops.yaml automate it
107	
108	Typing a long KMS ARN every encrypt is painful. SOPS can use a `.sops.yaml` config so path decides encryption rules.
109	
110	A typical production config:
111	
112	```yaml
113	# .sops.yaml
114	creation_rules:
115	  # When the filename matches secrets.dev.yaml, use the dev key
116	  - path_regex: '.*secrets\.dev\.yaml$'
117	    kms: 'arn:aws:kms:us-west-2:111111:alias/dev-sops'
118	
119	  # When the filename matches secrets.prod.yaml, use the prod key
120	  - path_regex: '.*secrets\.prod\.yaml$'
121	    kms: 'arn:aws:kms:ap-south-1:999999:alias/prod-sops'
122	```
123	
124	With this, developers only run `sops secrets.dev.yaml` and SOPS finds the matching KMS key. No manual intervention.
125	
126	## Automation: Helmfile and helm-secrets
127	
128	After manual encrypt/decrypt, integrate it into deploy. In the Helm ecosystem the recommended path is the **`helm-secrets`**** plugin** plus **Helmfile**。
129	
130	### How it works
131	
132	<hepta-color type="text" color="yellow">`helm-secrets`</hepta-color><hepta-color type="text" color="yellow"> is a Helm plugin that intercepts Helm. When it sees a values file encrypted by SOPS (often named </hepta-color><hepta-color type="text" color="yellow">`secrets.yaml`</hepta-color><hepta-color type="text" color="yellow">), it decrypts in memory before </hepta-color><hepta-color type="text" color="yellow">`helm install`</hepta-color><hepta-color type="text" color="yellow"> and passes plaintext to Helm.</hepta-color>
133	
134	<hepta-color type="text" color="yellow">**Key point: decrypted plaintext never lands on disk, which greatly cuts residual-secret risk.**</hepta-color>
135	
136	### Referencing in Helmfile
137	
138	In Helmfile, integrating `helm-secrets` is straightforward: list encrypted files under `secrets:`:
139	
140	```yaml
141	# helmfile.yaml
142	environments:
143	  prod:
144	    secrets:
145	      - secrets.prod.yaml # SOPS-encrypted file
146	
147	releases:
148	  - name: my-app
149	    chart: ./charts/my-app
150	    # Helmfile detects secrets.prod.yaml is encrypted and calls sops to decrypt
151	    values:
152	      - values.yaml
153	      - {{ .Values | toYaml | nindent 6 }}
154	```
155	
156	When you run `helmfile apply` the flow is:
157	
158	1. Helmfile reads `helmfile.yaml`。
159	
160	2. Sees the `secrets.prod.yaml` setting.
161	
162	3. Automatically calls `helm secrets` and, via `sops`, asks AWS KMS to decrypt.
163	
164	4. Merges decrypted content into the final deploy list.
165	
166	### CI/CD role permissions
167	
168	In CI/CD we do not put KMS keys in env vars (that would defeat the point). We use AWS service features:
169	
170	- **GitHub Actions / GitLab Runner**: get temporary IAM credentials via OIDC (such as `aws-actions/configure-aws-credentials`).
171	
172	- **Permission limits**: that IAM role must have `kms:Decrypt` on the matching KMS key.
173	
174	The CI/CD runner can then decrypt and deploy without holding any long-lived password.
175	
176	## Wrap-up: build your secrets defense
177	
178	We started from plaintext harm and why VCS and secrets are incompatible. With AWS KMS as root of trust, we moved key-protection pressure onto a mature cloud service.
179	
180	Then SOPS partial encryption keeps YAML structure and Git diffs readable. Finally the `helm-secrets` plugin plus Helmfile gives encrypt-at-rest, decrypt-just-in-time, never-land-on-disk deploys.
181	
182	KMS + SOPS + Helmfile is one of the most common GitOps secrets patterns in mid-to-large companies. It balances security, developer experience, and automation.
183	
184	> **Note**: SOPS solves storing secrets safely in Git, but it still injects them as Kubernetes **Secret objects**. If you need even higher security and do not want plaintext Secrets inside K8s either, the next section's **External Secrets architecture** is the endgame.
185	
186	---
187	
188	### Recap and next step
189	
190	This section moved us from plaintext crisis to encrypted management. You now have:
191	
192	- **Why not plaintext**: Git history is forever, and Helm commands can leak plaintext in the cluster.
193	
194	- **KMS core value**: keys stay in the cloud; IAM controls who can encrypt/decrypt; CloudTrail gives a full audit trail.
195	
196	- **SOPS strengths**: partial encryption keeps Git history structurally readable, with automatic multi-env key matching.
197	
198	- **Automation**: the `helm-secrets` plugin plus Helmfile decrypts at deploy time and leaves no trace.
199	
200	In production, SOPS is widely used for secrets tightly bound to Helm deploy params. For secrets that change often or should be centrally managed by AWS, we usually store them in **AWS Secrets Manager**。
201	
202	Next we upgrade the architecture: use **Secrets Store CSI Driver** (External Secrets) so Pods fetch keys from AWS Secrets Manager directly, with no ciphertext in Git. That is the last mile to fully stateless secret management.


## Note 69: How Helm Hooks Work

- id: `67a29044-2645-4d57-a716-ee0600c22cc8`
- last_edited: 2026-10-11T08:46:34.653Z
- whiteboards: DevOp All, Helm and Helmfile

card "How Helm Hooks Work" [67a29044-2645-4d57-a716-ee0600c22cc8] 112 lines
--- Databases ---
none
1	# How Helm Hooks Work
2	
3	Last lesson: `values.yaml` overrides so one Chart fits dev and prod. Rendering YAML and sending it is often not enough.
4	
5	<hepta-color type="text" color="yellow">You ship an app that changes the DB schema. New Pods must not connect until migration succeeds. Manual means scripts and humans. With </hepta-color><hepta-color type="text" color="yellow">**Helm Hooks**</hepta-color><hepta-color type="text" color="yellow"> that process control lives in the Chart.</hepta-color>
6	
7	## What a Hook is: a marked Kubernetes resource
8	
9	<hepta-color type="text" color="yellow">Core idea: </hepta-color><hepta-color type="text" color="yellow">**a Hook is not a new Kubernetes kind**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
10	
11	Underneath it is usually a standard **Job**, or a ConfigMap, Service, etc. It is a Hook because `metadata.annotations` tells Helm: do not treat this as a normal app piece — run it at a lifecycle point.
12	
13	<hepta-color type="text" color="yellow">On </hepta-color><hepta-color type="text" color="yellow">`install`</hepta-color><hepta-color type="text" color="yellow"> or </hepta-color><hepta-color type="text" color="yellow">`upgrade`</hepta-color><hepta-color type="text" color="yellow">, Helm scans </hepta-color><hepta-color type="text" color="yellow">`templates/`</hepta-color><hepta-color type="text" color="yellow">. A Hook-marked resource runs at the chosen time, must succeed, then the rest of the deploy continues.</hepta-color>
14	
15	### A typical Hook
16	
17	A DB-migration Job, annotated as a `pre-install` Hook:
18	
19	```yaml
20	apiVersion: batch/v1
21	kind: Job
22	metadata:
23	  name: {{ .Release.Name }}-db-migration
24	  annotations:
25	    # core Hook: run before resources install
26	    "helm.sh/hook": pre-install
27	    # delete the Job on success, keep the cluster clean
28	    "helm.sh/hook-delete-policy": hook-succeeded
29	spec:
30	  template:
31	    spec:
32	      containers:
33	        - name: migration
34	          image: my-app-migration:v1.2.0
35	          command: ["python", "manage.py", "migrate"]
36	      restartPolicy: OnFailure
37	```
38	
39	## Six lifecycle points
40	
41	Helm defines six moments to inject logic. Exact timing is how you design a solid deploy.
42	
43	| Hook timing | When | Typical use |
44	| --- | --- | --- |
45	| **pre-install** | After Chart resources render, <hepta-color type="text" color="yellow">before they go to the API Server.</hepta-color> | Create CRDs , run schema migrations, check external deps. |
46	| **post-install** | <hepta-color type="text" color="yellow">After every Chart resource exists on the API Server.</hepta-color> | <hepta-color type="text" color="yellow">Smoke tests </hepta-color> , tell monitoring deploy succeeded. |
47	| **pre-upgrade** | On `helm upgrade`, before resources update. | DB backup , pause scheduled jobs. |
48	| **post-upgrade** | After `helm upgrade` has updated all resources. | Flush caches, refresh LBs, verify upgraded features. |
49	| **pre-delete** | On `helm uninstall`, before resources are deleted. | Export persistent data , close external connections cleanly. |
50	| **post-delete** | After everything is gone from the cluster. | Clean leftover cloud records (LB), send delete-complete notices. |
51	
52	### `helm install` internal flow
53	
54	Where Hooks sit in `helm install`:
55	
56	1. **Render and validate**: Helm reads Chart + Values, renders YAML.
57	
58	2. <hepta-color type="text" color="yellow">**Run pre-install Hooks**</hepta-color><hepta-color type="text" color="yellow">: send every </hepta-color><hepta-color type="text" color="yellow">`pre-install`</hepta-color><hepta-color type="text" color="yellow"> resource to the API Server.</hepta-color>
59	
60	3. **Wait for Hooks**: Helm watches status (Jobs until `Succeeded`）。
61	
62	4. **Install real resources**: after Hooks succeed, install Deployment, Service, etc.
63	
64	5. **Run post-install Hooks**: after standard resources, run `post-install` tasks.
65	
66	6. **Return**: all steps including Hooks succeed → Release `deployed`。
67	
68	**Note**: Hook fail (Job non-zero exit) stops the rest. A strong fence against a broken config starting the real service.
69	
70	## Order: Hook Weight
71	
72	Many Hooks in one Chart: who first? ConfigMap before the Job that needs it.
73	
74	Use the `helm.sh/hook-weight` annotation.
75	
76	- **How**: integer (negative, 0, positive), as a string.
77	
78	- **Logic**: **smaller** runs earlier.
79	
80	- **Default**: omitted → `0`。
81	
82	```yaml
83	metadata:
84	  annotations:
85	    "helm.sh/hook": pre-install
86	    "helm.sh/hook-weight": "-5" # runs before weight 0
87	```
88	
89	<hepta-color type="text" color="yellow">Split init into small Jobs and run them in topological order.</hepta-color>
90	
91	## Cleanup: no Job landfill
92	
93	Hooks are often Jobs. Kubernetes keeps finished Jobs for logs. Frequent deploys pile them up.
94	
95	Automate with `helm.sh/hook-delete-policy`:
96	
97	1. <hepta-color type="text" color="yellow">**`before-hook-creation`**</hepta-color><hepta-color type="text" color="yellow"> (recommended): delete the old same-named Hook before starting the new one. Avoids name clashes.</hepta-color>
98	
99	2. <hepta-color type="text" color="yellow">**`hook-succeeded`**</hepta-color><hepta-color type="text" color="yellow">: delete only on success. Keep failures for logs.</hepta-color>
100	
101	3. <hepta-color type="text" color="yellow">**`hook-failed`**</hepta-color><hepta-color type="text" color="yellow">: delete only on fail (rare — we usually keep failed Jobs).</hepta-color>
102	
103	In practice combine policies, e.g.
104	`"helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded`。
105	
106	### Takeaway
107	
108	Hooks join ops steps to resource definitions. Runbook becomes executable code. CI/CD can be highly automatic.
109	
110	<hepta-color type="text" color="yellow">Always think </hepta-color><hepta-color type="text" color="yellow">**idempotency**</hepta-color><hepta-color type="text" color="yellow">. </hepta-color><hepta-color type="text" color="yellow">`upgrade`</hepta-color><hepta-color type="text" color="yellow"> may run many times. SQL migrations must be safe to repeat.</hepta-color>
111	
112


## Note 70: Minimal Difference Design Techniques

- id: `a95fb208-5b62-4c59-a89d-4984eb13aded`
- last_edited: 2026-10-11T08:42:10.693Z
- whiteboards: DevOp All, Helm and Helmfile

card "Minimal Difference Design Techniques" [a95fb208-5b62-4c59-a89d-4984eb13aded] 115 lines
--- Databases ---
none
1	# Minimal Difference Design Techniques
2	
3	Imagine you just inherited a project with 15 microservices, each with `dev`、`staging`、`prod` three environments. If you find that each environment's `values.yaml` is two hundred lines long, and 90% of the content is identical, how would you feel? When you need to change a global resource limit, do you have to search and replace across 45 files by hand? Worse, if you accidentally omit a production database password, the system only fails at runtime—that would be a disaster.
4	
5	In the previous part, we learned how to use the `-f` parameter to override values. This section goes further: how to design environment-specific `values.yaml` files so they stay minimal, yet still have strong error-proofing.
6	
7	## <hepta-color type="text" color="yellow">The required function: a safety gate for production</hepta-color>
8	
9	<hepta-color type="text" color="yellow">When writing a Helm Chart, some settings cannot be compromised—database passwords, API keys, or certificates. If they are missing, the app can still be deployed to Kubernetes, but Pods fail to start and fall into </hepta-color><hepta-color type="text" color="yellow">`CrashLoopBackOff`</hepta-color><hepta-color type="text" color="yellow">, or worse, fail silently with logic errors.</hepta-color>
10	
11	<hepta-color type="text" color="yellow">`required`</hepta-color><hepta-color type="text" color="yellow"> exists for this. It kicks in at </hepta-color><hepta-color type="text" color="yellow">**render time**</hepta-color><hepta-color type="text" color="yellow">. If the given value is missing or an empty string, Helm aborts rendering and throws an error, so incomplete YAML never reaches the cluster.</hepta-color>
12	
13	### Syntax and use cases
14	
15	`required` is typically used as part of a pipeline, with a clear error message:
16	
17	```yaml
18	# templates/deployment.yaml
19	apiVersion: apps/v1
20	kind: Deployment
21	metadata:
22	  name: {{ include "my-app.fullname" . }}
23	spec:
24	  template:
25	    spec:
26	      containers:
27	        - name: app
28	          env:
29	            - name: DB_PASSWORD
30	              value: {{ .Values.database.password | required "You must set database.password in values.yaml to deploy!" | quote }}
31	```
32	
33	This is more than a guardrail—it is **self-documenting**. When a new teammate installs your Chart and forgets the password, they see your own instructions, not a cryptic error.
34	
35	### Why this matters in production
36	
37	In production, security usually means "no defaults." You do not want someone carelessly using the development default password `password123`. With `required`, you can require that `values-prod.yaml` explicitly supplies a value from a secret manager (such as SOPS or AWS Secrets Manager).
38	
39	## <hepta-color type="text" color="yellow">The default function: the art of subtracting redundancy</hepta-color>
40	
41	<hepta-color type="text" color="yellow">If </hepta-color><hepta-color type="text" color="yellow">`required`</hepta-color><hepta-color type="text" color="yellow"> is a hard rule, then </hepta-color><hepta-color type="text" color="yellow">`default`</hepta-color><hepta-color type="text" color="yellow"> is a gentle fallback.</hepta-color>
42	
43	<hepta-color type="text" color="yellow">In multi-environment config, the goal is to record only the differences. If dev, test, and prod share the same image repository and only the tag differs, you should not repeat the path three times.</hepta-color>
44	
45	### How to cut boilerplate
46	
47	Consider this example: we want Pod resource limits, but they should be optional:
48	
49	```yaml
50	# templates/deployment.yaml
51	resources:
52	  requests:
53	    cpu: {{ .Values.resources.requests.cpu | default "100m" }}
54	    memory: {{ .Values.resources.requests.memory | default "256Mi" }}
55	```
56	
57	The benefit is that the user's `values-dev.yaml` can omit the `resources` block entirely, and Helm fills in `100m`. Only when production needs more performance do you write in `values-prod.yaml`:
58	
59	```yaml
60	# values-prod.yaml
61	resources:
62	  requests:
63	    cpu: "500m"
64	    memory: "1Gi"
65	```
66	
67	This turns `values.yaml` into an incremental changelog, not a full copy.
68	
69	### Advanced tip: tying to .Chart.AppVersion
70	
71	Another common pattern is linking `default` to Chart metadata. For example, by default you want the image tag to match `Chart.yaml` in `appVersion`:
72	
73	```yaml
74	image: {{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}
75	```
76	
77	Then, unless you need a canary test version in a specific environment, you do not even set `image.tag` in environment values—just update `Chart.yaml`.
78	
79	## Zero-config joining: Convention over Configuration
80	
81	Zero-config joining is a design philosophy. An ideal Chart should:
82	
83	1. **Be developer-friendly**: locally running `helm install my-app ./my-chart` (with no `-f`) should use defaults in `values.yaml` and the `default` function, and start on Minikube or Kind.
84	
85	2. **Be strict in production**: once you switch to production (`-f values-prod.yaml`), fields marked `required` force the deployer to confirm critical parameters.
86	
87	### Practice: environment-tiered settings
88	
89	You can design Values in three layers:
90	
91	| Layer | How | Purpose |
92	| --- | --- | --- |
93	| **Global defaults** | `values.yaml` and the `default` function | Define 90% of shared logic (Service type, health-check path, and so on). |
94	| **Environment diffs** | `values-dev.yaml` files | Describe only what is unique to that environment (Ingress domain, replica count). |
95	| **Safety checks** | `required` function | Guard environment-specific fields with no default (such as API keys). |
96	
97	With this layering, your Chart is like a precision instrument: it works out of the box, with safety switches on critical parts.
98	
99	## <hepta-color type="text" color="yellow">Pitfall: why overusing --set is dangerous</hepta-color>
100	
101	<hepta-color type="text" color="yellow">After learning override precedence, beginners often fall in love with the </hepta-color><hepta-color type="text" color="yellow">`---set`</hepta-color><hepta-color type="text" color="yellow"> flag. After all, </hepta-color><hepta-color type="text" color="yellow">`helm install --set replicas=5 ...`</hepta-color><hepta-color type="text" color="yellow"> looks fast and convenient. In production, though, </hepta-color><hepta-color type="text" color="yellow">**overusing **</hepta-color><hepta-color type="text" color="yellow">**`--set`**</hepta-color><hepta-color type="text" color="yellow">** is extremely risky.**</hepta-color>
102	
103	### <hepta-color type="text" color="yellow">1. No audit trail</hepta-color>
104	
105	<hepta-color type="text" color="yellow">In GitOps, every change should go through a Git commit. If you change replica count via </hepta-color><hepta-color type="text" color="yellow">`--set`</hepta-color><hepta-color type="text" color="yellow"> in CI/CD, that "5" lives only in CI logs, not in the repo. A month later, when someone asks why it became 5, nobody has an answer.</hepta-color>
106	
107	### <hepta-color type="text" color="yellow">2. The typo nightmare</hepta-color>
108	
109	<hepta-color type="text" color="yellow">`--set`</hepta-color><hepta-color type="text" color="yellow"> does not check that the key is valid. If you type </hepta-color><hepta-color type="text" color="yellow">`--set replcas=5`</hepta-color><hepta-color type="text" color="yellow"> (missing an i), Helm does not error—it quietly ignores the value. Without </hepta-color><hepta-color type="text" color="yellow">`required`</hepta-color><hepta-color type="text" color="yellow">, your Deployment stays at the default of 1 replica until traffic overwhelms it.</hepta-color>
110	
111	### <hepta-color type="text" color="yellow">3. No code review</hepta-color>
112	
113	<hepta-color type="text" color="yellow">YAML files can be reviewed; CLI argument strings cannot. Putting settings in files lets teammates see in a pull request: "Ah, we raised Staging DB memory this time."</hepta-color>
114	
115	<hepta-color type="text" color="yellow">**Best practice:**</hepta-color><hepta-color type="text" color="yellow"> Except for dynamic values injected by CI (current image tag or commit ID), all static config should live in </hepta-color><hepta-color type="text" color="yellow">`values-*.yaml`</hepta-color><hepta-color type="text" color="yellow"> files.</hepta-color>


## Note 71: Helm CLI Workflow

- id: `99ab3028-af9a-415d-92a1-ab7e1bd83055`
- last_edited: 2026-10-11T05:39:42.010Z
- whiteboards: DevOp All, Helm and Helmfile

card "Helm CLI Workflow" [99ab3028-af9a-415d-92a1-ab7e1bd83055] 154 lines
--- Databases ---
none
1	# Helm CLI Workflow
2	
3	Imagine Friday 5 p.m., last deploy of the week. You used `kubectl apply -f deployment.yaml` to update production, then alerts fire—the new version has a serious performance issue. You scramble for the previous YAML, but local files have been edited many times and Git commits are messy.
4	
5	<hepta-color type="text" color="yellow">That is the most fragile moment of managing K8s resources by hand. With Helm it takes one second:</hepta-color><hepta-color type="text" color="yellow">`helm rollback my-app 1`</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
6	
7	<hepta-color type="text" color="yellow">After Chart, Values, and Release mental models, this section covers the full Helm CLI flow. You will control the Release lifecycle with commands and see how Revision makes deploys safe and predictable.</hepta-color>
8	
9	## From kubectl to helm: a higher layer
10	
11	You already know `kubectl` well. With `kubectl apply` you operate resources—tell K8s to create a Deployment, Service, or Ingress. Like assembling furniture one board and one screw at a time.
12	
13	<hepta-color type="text" color="yellow">While </hepta-color><hepta-color type="text" color="yellow">`helm install`</hepta-color><hepta-color type="text" color="yellow"> operates packages. One command, Helm uses Chart (recipe) and Values (ingredients) to generate a related set of resources and manage them as one </hepta-color><hepta-color type="text" color="yellow">**Release**</hepta-color><hepta-color type="text" color="yellow">. That lift gives two core benefits:</hepta-color>
14	
15	1. <hepta-color type="text" color="yellow">**Atomicity**</hepta-color><hepta-color type="text" color="yellow">: a set of resources either fully deploys or is handled on failure, with no half-finished wreckage.</hepta-color>
16	
17	2. <hepta-color type="text" color="yellow">**Versioning**</hepta-color><hepta-color type="text" color="yellow">: every deploy leaves a fingerprint; you can inspect history or jump back.</hepta-color>
18	
19	## Release lifecycle: commands and state
20	
21	A Release in the cluster usually goes through install, observe, upgrade, rollback, and remove.
22	
23	### 1. Install: give it life
24	
25	The first time you deploy a Chart, use `install`.
26	
27	```bash
28	# syntax: helm install [RELEASE_NAME] [CHART_PATH] [FLAGS]
29	helm install my-api ./charts/web-app -f values-prod.yaml --namespace production
30	```
31	
32	This command does several things:
33	
34	- <hepta-color type="text" color="yellow">Reads templates in </hepta-color><hepta-color type="text" color="yellow">`./charts/web-app`</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
35	
36	- Uses values in `values-prod.yaml` to override defaults.
37	
38	- Creates all K8s resources in namespace `production`.
39	
40	- <hepta-color type="text" color="yellow">Creates a </hepta-color><hepta-color type="text" color="yellow">`my-api`</hepta-color><hepta-color type="text" color="yellow"> named </hepta-color><hepta-color type="text" color="yellow">**Revision 1**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
41	
42	### 2. Observe: know the current state
43	
44	After deploy, confirm Release health.
45	
46	- **`helm list`**: list all installed Releases in the current namespace so you know what is running.
47	
48	- **`helm status my-api`**: details for one Release, including last update time, status (such as `deployed` or `failed`), and Chart `NOTES.txt` (post-install notes).
49	
50	### 3. Upgrade: iterate
51	
52	To change image version, replicas, or any setting, use `upgrade`。
53	
54	```bash
55	helm upgrade my-api ./charts/web-app --set replicaCount=5
56	```
57	
58	Note that `upgrade` pushes the current Release to **Revision 2**. Even a tiny param change is recorded.
59	
60	### 4. Rollback: rewind time
61	
62	Helm's strongest feature. If Revision 2 is broken, send the Release back to Revision 1 immediately.
63	
64	```bash
65	helm rollback my-api 1
66	```
67	
68	<hepta-color type="text" color="yellow">**A key idea:**</hepta-color><hepta-color type="text" color="yellow"> Rollback actually creates a </hepta-color><hepta-color type="text" color="yellow">**new Revision**</hepta-color><hepta-color type="text" color="yellow">. After the command above you get </hepta-color><hepta-color type="text" color="yellow">**Revision 3**</hepta-color><hepta-color type="text" color="yellow">, and Revision 3 content matches Revision 1 exactly. </hepta-color><hepta-color type="text" color="yellow"> </hepta-color><hepta-color type="text" color="yellow">**This keeps version numbers monotonically increasing and easy to trace.**</hepta-color>
69	
70	### 5. Uninstall: clean up
71	
72	When the service is no longer needed:
73	
74	```bash
75	helm uninstall my-api
76	```
77	
78	<hepta-color type="text" color="yellow">This deletes all K8s resources for that Release and clears its record.</hepta-color>
79	
80	## Versioning core: Revision and History
81	
82	<hepta-color type="text" color="yellow">Why can Helm roll back? The secret is </hepta-color><hepta-color type="text" color="yellow">**Revision**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
83	
84	<hepta-color type="text" color="yellow">In Helm 3, every </hepta-color><hepta-color type="text" color="yellow">`install`</hepta-color><hepta-color type="text" color="yellow"> or </hepta-color><hepta-color type="text" color="yellow">`upgrade`</hepta-color><hepta-color type="text" color="yellow"> creates a new Revision, stored encrypted in a Kubernetes </hepta-color><hepta-color type="text" color="yellow">**Secret**</hepta-color><hepta-color type="text" color="yellow"> in the same namespace as the Release.  In other words, Helm state lives in the cluster itself; you need no extra database.</hepta-color>
85	
86	<hepta-color type="text" color="yellow">You can inspect these footprints with </hepta-color><hepta-color type="text" color="yellow">`helm history`</hepta-color><hepta-color type="text" color="yellow">:</hepta-color>
87	
88	```bash
89	helm history my-api
90	```
91	
92	The table looks like:
93	
94	| REVISION | UPDATED | STATUS | CHART | APP VERSION | DESCRIPTION |
95	| --- | --- | --- | --- | --- | --- |
96	| 1 | Fri Oct 20 10:00:00 2023 | superseded | web-app-1.0.0 | 3.0.0 | Install complete |
97	| 2 | Fri Oct 20 17:00:00 2023 | superseded | web-app-1.1.0 | 3.1.0 | Upgrade failed |
98	| 3 | Fri Oct 20 17:05:00 2023 | deployed | web-app-1.0.0 | 3.0.0 | Rollback to 1 |
99	
100	From `DESCRIPTION` and `STATUS` you can see at a glance that Revision 2 failed and current Revision 3 rolled back from 1. That transparency matters in DevOps: it removes guessing who changed what and when.
101	
102	## Dev debug tools: lint and --debug
103	
104	While writing Charts, besides install and preview, Helm gives two useful dev tools:
105	
106	- <hepta-color type="text" color="yellow">**`helm lint`**</hepta-color><hepta-color type="text" color="yellow">: static checker. It scans the Chart directory for YAML syntax, </hepta-color><hepta-color type="text" color="yellow">`Chart.yaml`</hepta-color><hepta-color type="text" color="yellow"> format, and Helm best practices. Put this command first in CI.</hepta-color>
107	
108	- <hepta-color type="text" color="yellow">**`--debug`**</hepta-color><hepta-color type="text" color="yellow">** flag**</hepta-color><hepta-color type="text" color="yellow">: global verbose mode. Add it after </hepta-color><hepta-color type="text" color="yellow">`install`</hepta-color><hepta-color type="text" color="yellow">、</hepta-color><hepta-color type="text" color="yellow">`upgrade`</hepta-color><hepta-color type="text" color="yellow"> or </hepta-color><hepta-color type="text" color="yellow">`template`</hepta-color><hepta-color type="text" color="yellow">. It prints richer errors and, even on a bad render, still dumps as much YAML as it can so you can find the broken template line.</hepta-color>
109	
110	---
111	
112	## Why preview matters: helm template vs helm install
113	
114	The most dangerous thing before production is running blindly. Helm gives a strong tool to see the future:`helm template`。
115	
116	### What is dry-run mode?
117	
118	`helm template` runs Go Template rendering **locally**. It reads Chart and Values, builds the final K8s YAML, and prints it to stdout, with no talk to the cluster.
119	
120	Compared with `helm install`:
121	
122	| Trait | `helm template` (preview) | `helm install` (deploy) |
123	| --- | --- | --- |
124	| **Where it runs** | <hepta-color type="text" color="yellow">Local</hepta-color> | Render locally, then send to the K8s cluster |
125	| **Network** | <hepta-color type="text" color="yellow">No cluster connection needed</hepta-color> | Must reach the K8s API Server |
126	| **Resource creation** | <hepta-color type="text" color="yellow">Creates nothing</hepta-color> | Actually creates Pod, Service, Ingress, etc. |
127	| **State record** | <hepta-color type="text" color="yellow">No Revision</hepta-color> | Creates a new Revision |
128	| **Main use** | <hepta-color type="text" color="yellow">**CI checks, debug, YAML syntax**</hepta-color> | Actually ship the app |
129	
130	### Why this matters for CI
131	
132	In modern GitOps or CI we usually run this at pull-request time `helm template`。
133	
134	- <hepta-color type="text" color="yellow">**Catch syntax errors**</hepta-color><hepta-color type="text" color="yellow">: if a template is wrong (missing a brace), </hepta-color><hepta-color type="text" color="yellow">`template`</hepta-color><hepta-color type="text" color="yellow"> errors and fails CI.</hepta-color>
135	
136	- **Preview the change**: save `template` output to a file and use `diff` to compare old vs new YAML. Reviewers can see: this change only adds an env var, no resource limits touched.
137	
138	## Best practice: preview then deploy
139	
140	To keep production stable, build this habit:
141	
142	1. **Local render preview**:
143	   Run `helm template my-release ./chart -f values.yaml > debug.yaml`. Open `debug.yaml` and check the generated resources.
144	
145	2. **Connected dry run**:
146	   To confirm the K8s server will accept the YAML (API version, quotas), use `helm install --dry-run --debug`. It sends rendered YAML for server validation without installing.
147	
148	3. **Real deploy**:
149	   When it looks good, run `helm install` or `helm upgrade`。
150	
151	4. **Verify status**:
152	   Use `helm status` and `helm history` to check the result.
153	
154	This layered filter catches most human errors (typos, bad config logic) before deploy．


## Note 72: Multi-Environment Override Mechanism

- id: `d71fc406-12f3-4d32-b021-fc2ccfc87998`
- last_edited: 2026-10-11T04:24:20.136Z
- whiteboards: DevOp All, Helm and Helmfile

card "Multi-Environment Override Mechanism" [d71fc406-12f3-4d32-b021-fc2ccfc87998] 72 lines
--- Databases ---
none
1	# Multi-Environment Override Mechanism
2	
3	we have covered `values.yaml` design principles: the Chart’s “API contract” or “default recipe.” In practice you almost never use the same settings in every environment. Dev may need one replica; production needs HA. Dev’s database is in-cluster; production talks to a managed cloud service.
4	
5	<hepta-color type="text" color="yellow">If you copy a full </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow"> per environment, you soon hit config drift—rename a default field in the Chart, forget ten env files, and deploy fails. Here we can use Helm’s override mechanism to manage those diffs cleanly and safely.</hepta-color>
6	
7	## Environment-specific Values files: escape YAML explosion
8	
9	Before Helm, multi-env usually meant copying lots of `deployment-dev.yaml`、`deployment-prod.yaml` files. That is hard to maintain and easy to get wrong. <hepta-color type="text" color="yellow">Helm’s idea: </hepta-color><hepta-color type="text" color="yellow">**split templates from values**</hepta-color><hepta-color type="text" color="yellow">, then split default values from environment diffs.</hepta-color>
10	
11	### <hepta-color type="text" color="yellow">Naming convention and management strategy</hepta-color>
12	
13	<hepta-color type="text" color="yellow">Helm does not require env file names, but this convention is widely used in DevOps:</hepta-color>
14	
15	- <hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow">: at the Chart root; defaults shared by every environment.</hepta-color>
16	
17	- <hepta-color type="text" color="yellow">`values-dev.yaml`</hepta-color><hepta-color type="text" color="yellow">: dev-only settings.</hepta-color>
18	
19	- <hepta-color type="text" color="yellow">`values-staging.yaml`</hepta-color><hepta-color type="text" color="yellow">: staging-only settings.</hepta-color>
20	
21	- <hepta-color type="text" color="yellow">`values-prod.yaml`</hepta-color><hepta-color type="text" color="yellow">: production-only settings.</hepta-color>
22	
23	These env files usually live with the Chart (if you own it) or in a separate config repo. On install, pass them with `-f` or `--values`:
24	
25	```bash
26	# Deploy to development
27	helm install my-app ./chart -f values-dev.yaml
28	
29	# Deploy to production
30	helm install my-app ./chart -f values-prod.yaml
31	```
32	
33	### “Record only the diffs”: DRY (Don't Repeat Yourself)
34	
35	This is the most important multi-env idea.<hepta-color type="text" color="yellow">** An env file should not be a full copy of **</hepta-color><hepta-color type="text" color="yellow">**`values.yaml`**</hepta-color><hepta-color type="text" color="yellow">**.**</hepta-color>
36	
37	If `values.yaml` is 100 lines and production only differs in `replicaCount` and `resources`, copying all 100 lines into `values-prod.yaml` means a later label-format change in `values.yaml` must be repeated in every env file.
38	
39	**The right way:** put only the keys you need to override in `values-prod.yaml`. Helm loads all of `values.yaml` first, then replaces matching keys from `values-prod.yaml`. Think inheritance and override in OOP.
40	
41	## Deep dive: Helm’s four-layer override priority
42	
43	When the same setting (for example `replicaCount`) appears in several places, which value wins? That order matters for debugging and CI/CD design.
44	
45	From lowest to highest priority:
46	
47	### Layer 1: Chart values.yaml (lowest)
48	
49	Chart-author defaults. The base so the Chart still runs in a basic, generic way if the user passes nothing extra.
50	
51	### Layer 2: Values from a parent Chart
52	
53	If this Chart is a subchart, the parent can override your defaults in its `values.yaml` under the subchart name. Coordinators of large projects can steer lower components from one place.
54	
55	### Layer 3: files passed with -f (--values)
56	
57	The main multi-env tool. CLI files override all earlier defaults.
58	
59	**Advanced: with several -f files, later wins.**
60	<hepta-color type="text" color="yellow"> Helm processes them left to right. For example:
61	</hepta-color><hepta-color type="text" color="yellow">`helm install my-app ./chart -f values.yaml -f values-prod.yaml`</hepta-color><hepta-color type="text" color="yellow">
62	If both files set </hepta-color><hepta-color type="text" color="yellow">`image.tag`</hepta-color><hepta-color type="text" color="yellow">, </hepta-color><hepta-color type="text" color="yellow">`values-prod.yaml`</hepta-color><hepta-color type="text" color="yellow"> wins. Useful for shared global settings vs app-specific settings.</hepta-color>
63	
64	### Layer 4: --set values (highest)
65	
66	<hepta-color type="text" color="yellow">The last word. CLI </hepta-color><hepta-color type="text" color="yellow">`--set`</hepta-color><hepta-color type="text" color="yellow"> has highest priority and forcibly rewrites values from every earlier file.</hepta-color>
67	
68	```bash
69	# Even if values-prod.yaml has replicas=3
70	# this command deploys replicas=5
71	helm install my-app ./chart -f values-prod.yaml --set replicaCount=5
72	```


## Note 73: values.yaml Design Principles

- id: `4e08a171-16da-4f70-a7e7-c139d7619d42`
- last_edited: 2026-10-11T04:21:48.107Z
- whiteboards: DevOp All, Helm and Helmfile

card "values.yaml Design Principles" [4e08a171-16da-4f70-a7e7-c139d7619d42] 151 lines
--- Databases ---
none
1	# values.yaml Design Principles
2	
3	When you write a Helm Chart for a complex app, you face a dilemma: should every YAML field become a variable? Too few and users say the Chart is inflexible; too many and `values.yaml` becomes thousands of lines and nobody wants to open it.
4	
5	Imagine designing a microwave panel. Good design lets people pick “popcorn” or “30 seconds,” not magnetron wattage or fan RPM. In Helm, `values.yaml` is that panel.
6	
7	## values.yaml as an API contract
8	
9	<hepta-color type="text" color="yellow">In software we talk about an API contract. You call a function without reading its 500 lines; you only need inputs and outputs.</hepta-color>
10	
11	<hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow"> is that contract for a Helm Chart.</hepta-color>
12	
13	### Defaults as documentation
14	
15	<hepta-color type="text" color="yellow">A well-designed </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow"> should let users configure the app without reading any YAML under </hepta-color><hepta-color type="text" color="yellow">`templates/`</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
16	
17	Defaults should not only make it run; they should guide. For example:
18	
19	- Do not leave blanks: if a field is optional, give a sensible default and comment it.
20	
21	- Use comments: purpose, allowed values, and what changing it does.
22	
23	<hepta-color type="text" color="yellow">When users open your Chart they usually look at </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow"> first. Messy structure and no comments force them into Go templates. That is a burden, and it means the encapsulation failed.</hepta-color>
24	
25	### Hide implementation details
26	
27	As Chart author you own Kubernetes complexity. Users may only want “turn HPA on,” not whether HPA’s `apiVersion` should be `autoscaling/v2` or `v2beta2`。
28	
29	<hepta-color type="text" color="yellow">Hide infra details that are not business logic inside </hepta-color><hepta-color type="text" color="yellow">`templates/`</hepta-color><hepta-color type="text" color="yellow">, and expose semantic switches in </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow">. That is what separates a good Chart from an average one.</hepta-color>
30	
31	## Semantic names: the community language
32	
33	<hepta-color type="text" color="yellow">When naming fields in </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow">, extra creativity is not helpful. Follow Helm community convention to lower the learning curve.</hepta-color>
34	
35	### Why follow convention?
36	
37	If every restaurant called water something else—“transparent liquid,” “H2O,” “source of life”—you would be exhausted.
38	
39	In Helm, most official and mature community Charts (Bitnami) share an informal naming scheme. Seeing `image.repository`, users know that is the Docker image path.
40	
41	### Common naming cheat sheet
42	
43	Prefer these structures when you design:
44	
45	- **Image settings**：
46	
47	  ```yaml
48	  image:
49	    repository: my-app
50	    tag: "1.0.0"
51	    pullPolicy: IfNotPresent
52	  ```
53	
54	- **Resource limits**：
55	
56	  ```yaml
57	  resources:
58	    requests:
59	      cpu: 100m
60	      memory: 128Mi
61	    limits:
62	      cpu: 200m
63	      memory: 256Mi
64	  ```
65	
66	- **Service settings**：
67	
68	  ```yaml
69	  service:
70	    type: ClusterIP
71	    port: 80
72	  ```
73	
74	<hepta-color type="text" color="yellow">Follow the convention and the next engineer does not relearn your Chart. That intuition is professionalism.</hepta-color>
75	
76	## Structure: balance nesting and readability
77	
78	Beginners often err here. Some flatten `values.yaml` (everything at the top); others nest as deep as Kubernetes objects.
79	
80	### The deep-nesting trap
81	
82	If `values.yaml` mirrors Kubernetes resources, you get something like:
83	
84	```yaml
85	# Bad example: too much nesting
86	deployment:
87	  spec:
88	    template:
89	      spec:
90	        containers:
91	          main:
92	            image:
93	              tag: v1
94	```
95	
96	In the template you must write `{{ .Values.deployment.spec.template.spec.containers.main.image.tag }}`.
97	Problems:
98	
99	1. **Hard to read**: templates get long and error-prone.
100	
101	2. **Fragile**: rename any intermediate level and every template reference breaks.
102	
103	3. **Hard to override**: a `--set` override needs a very long path.
104	
105	### Recommended: group by logic, keep it moderately flat
106	
107	<hepta-color type="text" color="yellow">Aim for </hepta-color><hepta-color type="text" color="yellow">**2 to 3 levels**</hepta-color><hepta-color type="text" color="yellow">, grouped by logical function, not by K8s resource kind.</hepta-color>
108	
109	<hepta-color type="text" color="yellow">**Recommended:**</hepta-color>
110	
111	```yaml
112	# Good example: clear logic, moderate structure
113	image:
114	  repository: my-app
115	  tag: v1
116	
117	replicaCount: 3
118	
119	service:
120	  port: 80
121	```
122	
123	Related knobs stay together (everything about image), without pointless nesting.
124	
125	## User view: which parameters should be visible
126	
127	<hepta-color type="text" color="yellow">The philosophy of </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow">: “Let users change what they care about; hide internals.”</hepta-color>
128	
129	<hepta-color type="text" color="yellow">What do users care about? It depends on the app.</hepta-color>
130	
131	### Must-have knobs
132	
133	Most production apps need at least:
134	
135	- <hepta-color type="text" color="yellow">**Environment diffs**</hepta-color><hepta-color type="text" color="yellow">: database connection strings, API endpoints.</hepta-color>
136	
137	- <hepta-color type="text" color="yellow">**Resources**</hepta-color><hepta-color type="text" color="yellow">: CPU/Memory Requests and Limits.</hepta-color>
138	
139	- <hepta-color type="text" color="yellow">**Scale**</hepta-color><hepta-color type="text" color="yellow">: replicaCount, whether HPA is on.</hepta-color>
140	
141	- <hepta-color type="text" color="yellow">**Security**</hepta-color><hepta-color type="text" color="yellow">: ServiceAccount name, whether a PullSecret is needed.</hepta-color>
142	
143	### Example of hiding internals
144	
145	Suppose the app needs a specific `labels` so Prometheus can scrape metrics. Do not make users type those labels in `values.yaml`. Generate them in `templates/` with `_helpers.tpl`, and at most expose a `values.yaml` `metrics.enabled: true` switch.
146	
147	**Think: ** if the Service can only start on port 8080, should that port be a variable?
148	
149	- If the Chart is internal to one service, you can hard-code it in the template.
150	
151	- If it is a general Chart for the community, expose it—infra rules differ by company.


## Note 74: Interpolation and Built-in Objects

- id: `246adee3-cebc-41bb-aa8e-210dd6e57c53`
- last_edited: 2026-10-11T04:14:28.531Z
- whiteboards: DevOp All, Helm and Helmfile

card "Interpolation and Built-in Objects" [246adee3-cebc-41bb-aa8e-210dd6e57c53] 33 lines
--- Databases ---
none
1	# Interpolation and Built-in Objects
2	
3	We already saw a Helm Chart directory layout, and that the `templates/` directory holds Kubernetes manifests. Open those files and they are not fully standard YAML—<hepta-color type="text" color="yellow">they are full of double braces </hepta-color><hepta-color type="text" color="yellow">`{{ }}`</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
4	
5	<hepta-color type="text" color="yellow">That is Helm's real core: </hepta-color><hepta-color type="text" color="yellow">**Go Template**</hepta-color><hepta-color type="text" color="yellow">. It turns static YAML into a live template driven by logic, environment, and parameters. If K8s YAML is a handwritten letter, Go Template is a mail merge: every "Dear \[Name]" is filled in before send.</hepta-color>
6	
7	## Double braces `{{ }}`: template-action boundaries
8	
9	In Helm templates, all dynamic logic lives inside double braces `{{ }}`. Those braces are a boundary: outside is static text; inside is code for the renderer.
10	
11	In practice that means:
12	
13	1. **Interpolation**: the simplest use is stuffing a variable into YAML. For example `replicas: {{ .Values.replicaCount }}`。
14	
15	2. **Control flow**: conditionals (if/else) and loops (range), covered later.
16	
17	3. **Placeholder**: the action's result replaces the entire `{{ ... }}` block.
18	
19	<hepta-color type="text" color="yellow">A common detail is </hepta-color><hepta-color type="text" color="yellow">**whitespace control**</hepta-color><hepta-color type="text" color="yellow">. You may see </hepta-color><hepta-color type="text" color="yellow">`{{-`</hepta-color><hepta-color type="text" color="yellow"> or </hepta-color><hepta-color type="text" color="yellow">`-}}`</hepta-color><hepta-color type="text" color="yellow"> (with a minus). That trims whitespace (newlines or spaces) on that side. Critical for YAML indentation. For now, focus on data sources inside the braces.</hepta-color>
20	
21	## Built-in objects: your data arsenal
22	
23	When Helm renders, it injects predefined data objects. They hang off the root scope, written as a dot `.`. All built-ins start with a dot, which is why you always see `.Values` or `.Release`。
24	
25	Let's unpack the four core built-ins.
26	
27	<hepta-mention type="card" id="ec7be789-679a-4187-b538-7311655f9a62">.Values: the user's settings</hepta-mention>
28	
29	<hepta-mention type="card" id="6f66a535-cdc9-46e1-908c-4449f6a0529b">.Release: live data for this deploy</hepta-mention>
30	
31	<hepta-mention type="card" id="2df5fb53-3432-441d-b98f-518046e720df">.Chart: Chart.yaml metadata</hepta-mention>
32	
33	<hepta-mention type="card" id="7815a125-feac-46d2-a5f7-cf212e5934ad">.Template: context for the current template file</hepta-mention>


## Note 75: .Template: context for the current template file

- id: `7815a125-feac-46d2-a5f7-cf212e5934ad`
- last_edited: 2026-10-11T04:14:28.512Z
- whiteboards: DevOp All, Helm and Helmfile

card ".Template: context for the current template file" [7815a125-feac-46d2-a5f7-cf212e5934ad] 11 lines
--- Databases ---
none
1	# `.Template`: context for the current template file
2	
3	When a Chart grows to dozens of templates, `.Template` becomes useful.
4	
5	- **`.Template.Name`**: path of the template being rendered (relative to Chart root). Example:`mychart/templates/deployment.yaml`。
6	
7	- **`.Template.BasePath`**: path of the current Chart `templates` directory.
8	
9	Handy for debug or auto-generated comments. For example, at the top of the YAML:
10	`# Source: {{ .Template.Name }}`
11	Then when you run `helm template` on a huge YAML stream, you can see which source file rendered each resource.


## Note 76: .Chart: Chart.yaml metadata

- id: `2df5fb53-3432-441d-b98f-518046e720df`
- last_edited: 2026-10-11T04:14:16.068Z
- whiteboards: DevOp All, Helm and Helmfile

card ".Chart: Chart.yaml metadata" [2df5fb53-3432-441d-b98f-518046e720df] 18 lines
--- Databases ---
none
1	# `.Chart`: Chart.yaml metadata
2	
3	This object lets templates read fields from `Chart.yaml`.
4	
5	- **`.Chart.Name`**: Chart name.
6	
7	- **`.Chart.Version`**: Chart package version.
8	
9	- **`.Chart.AppVersion`**: application version.
10	
11	**Key distinction: Version vs AppVersion**
12	Beginners mix these up constantly.
13	
14	- `version` is the recipe (Chart) version. Change a template label or fix a bug, bump `version`。
15	
16	- `appVersion` is the ingredient (app) version, usually the Docker image tag. Code goes v1 to v2 but Chart structure is unchanged: only update `appVersion`。
17	
18	In templates we often use `.Chart.AppVersion` as the default container image tag.


## Note 77: .Release: live data for this deploy

- id: `6f66a535-cdc9-46e1-908c-4449f6a0529b`
- last_edited: 2026-10-11T04:13:55.011Z
- whiteboards: DevOp All, Helm and Helmfile

card ".Release: live data for this deploy" [6f66a535-cdc9-46e1-908c-4449f6a0529b] 13 lines
--- Databases ---
none
1	# `.Release`: live data for this deploy
2	
3	`.Values` is what you fill in; `.Release` is generated by Helm at deploy time.
4	
5	- **`.Release.Name`**: the Release name for this install. If you run `helm install my-ghost bitnami/ghost`, then `.Release.Name` is `my-ghost`. Essential for globally unique names (e.g. `my-ghost-deployment`).
6	
7	- **`.Release.Namespace`**: the Namespace this Release is installed into. Resources often inherit namespace, but an explicit value helps when wiring across resources (e.g. Service to Ingress).
8	
9	- **`.Release.IsInstall`**** and ****`.Release.IsUpgrade`**: two booleans that make the template context-aware.
10	
11	  - *Example*: run a DB init Job only on first install (IsInstall); skip it on later upgrades.
12	
13	- **`.Release.Service`**: always `Helm`. Often used as the `heritage` label so you can see who manages the resource.


## Note 78: .Values: the user's settings

- id: `ec7be789-679a-4187-b538-7311655f9a62`
- last_edited: 2026-10-11T04:13:42.324Z
- whiteboards: DevOp All, Helm and Helmfile

card ".Values: the user's settings" [ec7be789-679a-4187-b538-7311655f9a62] 17 lines
--- Databases ---
none
1	# `.Values`: the user's settings
2	
3	This is the object you use most. It maps to the `values.yaml` file, plus anything you pass on the CLI with `-f` or `--set`.
4	
5	<hepta-color type="text" color="yellow">`.Values`</hepta-color><hepta-color type="text" color="yellow"> is nested. If your </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow"> looks like this:</hepta-color>
6	
7	```yaml
8	image:
9	  repository: nginx
10	  tag: "1.21.0"
11	```
12	
13	<hepta-color type="text" color="yellow">In the template you walk it with dot notation:</hepta-color><hepta-color type="text" color="yellow">`{{ .Values.image.repository }}`</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
14	
15	**Why it matters:**
16	
17	 `.Values` separates code from config. The template defines structure (e.g. an image path goes here); `.Values` supplies the values. One Chart can use `image: my-app:dev` in dev and `image: my-app:v1.0.0` in prod, without touching any file under `templates/`.


## Note 79: Anatomy of templates/ Directory

- id: `5eb8949c-f303-4ca0-94ec-b37d2bc16f67`
- last_edited: 2026-10-11T04:02:33.733Z
- whiteboards: DevOp All, Helm and Helmfile

card "Anatomy of templates/ Directory" [5eb8949c-f303-4ca0-94ec-b37d2bc16f67] 162 lines
--- Databases ---
none
1	# Anatomy of `templates/` Directory
2	
3	If you have ever maintained resource lists across many Kubernetes clusters by hand, you know the fear of changing one label and search-replacing it in a dozen YAML files. In Kubernetes, YAML is static and rigid; real needs are dynamic. That is why the `templates/` directory exists: it turns rigid manifests into a flexible factory. You no longer write dead manifests; you design a blueprint that generates resources from input Values.
4	
5	## `templates/`: the K8s resource factory
6	
7	<hepta-color type="text" color="yellow">In a standard Helm Chart, </hepta-color><hepta-color type="text" color="yellow">`templates/`</hepta-color><hepta-color type="text" color="yellow"> is the most active directory. It is the Helm render engine's workspace, holding every resource you want to deploy to the cluster.</hepta-color>
8	
9	### Typical files and their jobs
10	
11	Kubernetes does not care what you name YAML files as long as the content is valid, but the Helm community has a conventional layout. Opening `templates/` you usually see:
12	
13	1. **`deployment.yaml`**: how the app runs (replicas, container image, update strategy). The Chart's core.
14	
15	2. **`service.yaml`**: how to reach those Pods (ClusterIP, NodePort, or LoadBalancer).
16	
17	3. **`ingress.yaml`**: how external traffic enters the cluster, usually host and path mapping.
18	
19	4. **`hpa.yaml`** (Horizontal Pod Autoscaler): scale replicas from CPU or memory load.
20	
21	5. <hepta-color type="text" color="yellow">**`serviceaccount.yaml`**</hepta-color><hepta-color type="text" color="yellow">: identity for Pods talking to the Kubernetes API.</hepta-color>
22	
23	6. **`configmap.yaml / secret.yaml`**: config and sensitive data.
24	
25	<hepta-color type="text" color="yellow">These files look like standard Kubernetes YAML, but in Helm they are templated. Hard-coded values are now wrapped in </hepta-color><hepta-color type="text" color="yellow">`{{ }}`</hepta-color><hepta-color type="text" color="yellow"> double braces.</hepta-color>
26	
27	### Static YAML vs. templated YAML
28	
29	A concrete example. Left: a standard Deployment you already know. Right: the Helm version:
30	
31	| Standard Kubernetes Deployment | Helm templated Deployment |
32	| --- | --- |
33	| `name: my-app-v1` | `name: {{ .Release.Name }}` |
34	| `replicas: 3` | `replicas: {{ .Values.replicaCount }}` |
35	| `image: nginx:1.21` | `image: {{ .Values.image.repository }}:{{ .Values.image.tag }}` |
36	| `env: "production"` | `env: {{ .Values.envName }}` |
37	
38	The shift is about **separation of concerns**：
39	
40	- **Template**: structure and logic. For example, a Deployment with 3 replicas whose image name comes from a variable.
41	
42	- <hepta-color type="text" color="yellow">**Values**</hepta-color><hepta-color type="text" color="yellow">: concrete data. For example, in dev the image version is </hepta-color><hepta-color type="text" color="yellow">`latest`</hepta-color><hepta-color type="text" color="yellow">; in production it is </hepta-color><hepta-color type="text" color="yellow">`v1.2.0`</hepta-color><hepta-color type="text" color="yellow">」。</hepta-color>
43	
44	<hepta-color type="text" color="yellow">On install, Helm combines the two and produces the YAML sent to the API Server.</hepta-color>
45	
46	## `_helpers.tpl`: the Chart's private toolbox
47	
48	In `templates/` one file name is special—it starts with an underscore:`_helpers.tpl`。
49	
50	In Helm, **files starting with an underscore are not rendered as Kubernetes resources**. A standard Service definition inside one is ignored and never deployed. So why does it exist?
51	
52	### Why helpers?
53	
54	Imagine an app named `my-app`. To avoid name clashes you usually combine Release name and Chart name (for example `prod-my-app-web`). Per Kubernetes best practice, every resource should also carry a standard set of labels, including `app.kubernetes.io/name`、`app.kubernetes.io/instance` and so on.
55	
56	If the Chart has 10 resource files, would you repeat that naming logic and label list in 10 places?
57	
58	That is the stage for `_helpers.tpl`. It holds **named templates**. Think of them as functions or variable definitions for logic reused many times.
59	
60	### How it works
61	
62	In `_helpers.tpl` you use the `define` keyword to define a snippet:
63	
64	```yaml
65	{{- define "my-chart.labels" -}}
66	app.kubernetes.io/name: {{ .Chart.Name }}
67	app.kubernetes.io/instance: {{ .Release.Name }}
68	{{- end -}}
69	```
70	
71	Then in `deployment.yaml` or `service.yaml` you call it with `include`:
72	
73	```yaml
74	metadata:
75	  labels:
76	    {{- include "my-chart.labels" . | nindent 4 }}
77	```
78	
79	**Two points to note:**
80	
81	1. **Why ****`include`**** instead of ****`template`****？** Both can include a snippet, but `include` is a powerful Helm function: <hepta-color type="text" color="yellow">it lets you pipe the output, such as </hepta-color><hepta-color type="text" color="yellow">`nindent 4`</hepta-color><hepta-color type="text" color="yellow"> above. That is critical for YAML indent; we will cover it in the syntax section.</hepta-color>
82	
83	2. <hepta-color type="text" color="yellow">**What is the trailing dot (**</hepta-color><hepta-color type="text" color="yellow">**`.`**</hepta-color><hepta-color type="text" color="yellow">**)?**</hepta-color><hepta-color type="text" color="yellow"> In Helm templates the dot is the current scope. Passing the dot hands all </hepta-color><hepta-color type="text" color="yellow">`.Values`</hepta-color><hepta-color type="text" color="yellow">、</hepta-color><hepta-color type="text" color="yellow">`.Chart`</hepta-color><hepta-color type="text" color="yellow"> objects to the helper so it can work.</hepta-color>
84	
85	## `NOTES.txt` and Dependency Management
86	
87	`NOTES.txt`: the Chart's last mile
88	
89	After a successful `helm install` or `helm upgrade`, the terminal prints a block of text. It is not random — it comes from `templates/NOTES.txt`。
90	
91	<hepta-color type="text" color="yellow">Special: it lives under </hepta-color><hepta-color type="text" color="yellow">`templates/`</hepta-color><hepta-color type="text" color="yellow"> but it </hepta-color><hepta-color type="text" color="yellow">**does not**</hepta-color><hepta-color type="text" color="yellow"> become a Kubernetes object and is never sent to the API Server. It is only for humans.</hepta-color>
92	
93	### Why it matters
94	
95	1. **Lower the bar**: after install, the first question is usually "how do I reach this service?"
96	
97	2. <hepta-color type="text" color="yellow">**Dynamic content**</hepta-color><hepta-color type="text" color="yellow">：</hepta-color><hepta-color type="text" color="yellow">`NOTES.txt`</hepta-color><hepta-color type="text" color="yellow"> also supports Go templates. You can give different guidance based on </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
98	
99	### Example: dynamic guidance
100	
101	Suppose the Chart can expose via Ingress or via `port-forward`. A well-designed `NOTES.txt` looks like this:
102	
103	```text
104	Thanks for installing {{ .Chart.Name }}!
105	
106	Your Release name is: {{ .Release.Name }}
107	
108	{{- if .Values.ingress.enabled }}
109	1. Reach the service at:
110	   http://{{ .Values.ingress.host }}
111	{{- else }}
112	1. The service is cluster-internal only.
113	2. For local access:
114	   kubectl port-forward svc/{{ include "my-chart.fullname" . }} 8080:80
115	   then visit: http://127.0.0.1:8080
116	{{- end }}
117	
118	{{- if .Values.auth.enabled }}
119	3. Initial admin password:
120	   kubectl get secret {{ include "my-chart.fullname" . }}-auth -o jsonpath="{.data.admin-password}" | base64 --decode
121	{{- end }}
122	```
123	
124	The script shows different steps from the user's config (Ingress on? auth on?). Deploy feels obvious. No long `values.yaml` or guessing Secret names.
125	
126	## Helm render flow: from directory to API
127	
128	With file roles clear, you may wonder: when you run `helm install`, what does Helm actually do?
129	
130	### Scan and merge
131	
132	On install or upgrade, Helm scans everything under `templates/`. It scans top-level files and recurses into subdirectories if you group templates there.
133	
134	Note that <hepta-color type="text" color="yellow">Helm rendering is </hepta-color><hepta-color type="text" color="yellow">**holistic**</hepta-color><hepta-color type="text" color="yellow">. It loads every template into memory and renders with your Values. The result is not 10 separate files, but one </hepta-color><hepta-color type="text" color="yellow">**long YAML stream**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
135	
136	### YAML stream
137	
138	In Kubernetes you can put many resources in one file, separated by `---`. Helm uses that. It concatenates rendered `deployment.yaml`、`service.yaml` and the rest, separated by `---`.
139	
140	<hepta-color type="text" color="yellow">That stream is sent once to the Kubernetes API Server, which parses it and handles each resource by </hepta-color><hepta-color type="text" color="yellow">`kind`</hepta-color><hepta-color type="text" color="yellow"> (for example Deployment, Service).</hepta-color>
141	
142	<hepta-color type="text" color="yellow">That is why Helm does not make you specify install order. The API Server can handle dependencies (</hepta-color><hepta-color type="text" color="yellow">**though Helm also has a fixed sort: Namespace first, then ServiceAccount, then Deployment, so base resources exist first)**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
143	
144	### Debug superpower: helm template
145	
146	Because render is automatic and complex, a growing Chart is hard to inspect by eye. Then `helm template` is your friend.
147	
148	```bash
149	helm template my-release ./my-chart
150	```
151	
152	<hepta-color type="text" color="yellow">This command runs full render logic but </hepta-color><hepta-color type="text" color="yellow">**does not**</hepta-color><hepta-color type="text" color="yellow"> connect to the cluster or create a Release. It prints that long YAML stream in the terminal. It is the tool you use most while developing a Chart, to confirm </hepta-color><hepta-color type="text" color="yellow">`_helpers.tpl`</hepta-color><hepta-color type="text" color="yellow"> references, variable injection, and indent are correct.</hepta-color>
153	
154	## Why separation of concerns matters
155	
156	Again: templates define structure, Values provide data. When designing files under `templates/`, ask:**What never changes? What changes per environment?**
157	
158	- <hepta-color type="text" color="yellow">**Stable (stay in the template)**</hepta-color><hepta-color type="text" color="yellow">: container start command, port definitions, label keys, liveness probe path.</hepta-color>
159	
160	- <hepta-color type="text" color="yellow">**Variable (extract to Values)**</hepta-color><hepta-color type="text" color="yellow">: replica count, image tag, env vars (API keys), Ingress hostname, resource limits.</hepta-color>
161	
162	This makes the Chart highly portable. You can give the same Chart to dev, test, and ops; they only supply their own `values.yaml` and never touch your templates. That is how Helm solves YAML explosion.


## Note 80: Chart.yaml in depth: the metadata brain

- id: `ce934226-23d4-474a-b0b0-472f64fb9260`
- last_edited: 2026-10-11T03:49:19.862Z
- whiteboards: DevOp All, Helm and Helmfile

card "Chart.yaml in depth: the metadata brain" [ce934226-23d4-474a-b0b0-472f64fb9260] 78 lines
--- Databases ---
none
1	# Chart.yaml in depth: the metadata brain
2	
3	`Chart.yaml` is the Chart's core definition file in YAML. It sets basic attributes. Here is a standard `Chart.yaml` example:
4	
5	```yaml
6	apiVersion: v2
7	name: my-application
8	description: A sample Chart for deploying a microservice
9	type: application
10	version: 1.2.0
11	appVersion: "3.0.0"
12	maintainers:
13	  - name: tomtom
14	    email: tomtom@example.com
15	```
16	
17	### Core fields
18	
19	1. **apiVersion**: 
20	   The current standard is `v2`. That means the Chart is for Helm 3. Early Helm 2 used `v1`, but after Helm 3 removed Tiller and simplified the architecture, `v2` became the mainstream. If you see `v1`, it is usually a leftover from the old era.
21	
22	2. <hepta-color type="text" color="yellow">**name**</hepta-color>: 
23	   <hepta-color type="text" color="yellow">The Chart name. It usually matches the directory name. In the Helm ecosystem, name is the first identity check.</hepta-color>
24	
25	3. **type**: 
26	   <hepta-color type="text" color="yellow">Easy to skip, but important. Two options:</hepta-color>
27	
28	   - <hepta-color type="text" color="yellow">**application**</hepta-color><hepta-color type="text" color="yellow"> (default): a complete application that can be deployed to a Kubernetes cluster (Deployment, Service, and so on).</hepta-color>
29	
30	   - <hepta-color type="text" color="yellow">**library**</hepta-color><hepta-color type="text" color="yellow">: a library Chart. It deploys no resources; it only provides reusable template snippets for other Charts. Very useful in large companies to unify labels or security settings.</hepta-color>
31	
32	4. <hepta-color type="text" color="yellow">**version**</hepta-color><hepta-color type="text" color="yellow">: 
33	   This is the </hepta-color><hepta-color type="text" color="yellow">**Chart's own version**</hepta-color><hepta-color type="text" color="yellow">. It must follow </hepta-color><hepta-color type="text" color="yellow">**Semantic Versioning 2 (SemVer 2)**</hepta-color><hepta-color type="text" color="yellow"> (for example 1.2.0). Whenever you change template logic, add a variable, or adjust defaults, bump this version.</hepta-color> 
34	
35	5. **appVersion**: 
36	   This is the <hepta-color type="text" color="yellow">**version of the app inside**</hepta-color>. It does not have to follow SemVer .<hepta-color type="text" color="yellow">It usually matches your Docker Image Tag (for example "v3.0.0-beta" or "sha256:abcd")</hepta-color>. It tells users which version of the app (contents) this toolbox (Chart) holds.
37	
38	+ ## Version design: two versions in parallel
39	
40	  You may ask: why two version numbers? Isn't that extra trouble?
41	
42	  This is one of Helm's sharpest separations of power. A real production scene:
43	
44	  ### Scene A: app updates, Chart logic unchanged
45	
46	  Your product team ships `app:v2.1.0`, fixing a code bug.
47	
48	  - **What you do**: change `Chart.yaml` in `appVersion: "2.1.0"`。
49	
50	  - <hepta-color type="text" color="yellow">**Version change**</hepta-color><hepta-color type="text" color="yellow">：</hepta-color><hepta-color type="text" color="yellow">`version`</hepta-color><hepta-color type="text" color="yellow"> may go from </hepta-color><hepta-color type="text" color="yellow">`1.0.0`</hepta-color><hepta-color type="text" color="yellow"> to </hepta-color><hepta-color type="text" color="yellow">`1.0.1`</hepta-color><hepta-color type="text" color="yellow"> (the Chart contents changed), and </hepta-color><hepta-color type="text" color="yellow">`appVersion`</hepta-color><hepta-color type="text" color="yellow"> becomes </hepta-color><hepta-color type="text" color="yellow">`2.1.0`</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
51	
52	  - <hepta-color type="text" color="yellow">**Meaning**</hepta-color><hepta-color type="text" color="yellow">: the deploy method did not change; only the contents did.</hepta-color>
53	
54	  ### Scene B: app unchanged, deploy logic updates
55	
56	  The app is still `app:v2.1.0`, but ops finds CPU limits too low, or needs a new sidecar to collect logs.
57	
58	  - <hepta-color type="text" color="yellow">**What you do**</hepta-color><hepta-color type="text" color="yellow">: edit </hepta-color><hepta-color type="text" color="yellow">`templates/deployment.yaml`</hepta-color><hepta-color type="text" color="yellow"> to add a sidecar, and adjust default resource limits in </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
59	
60	  - <hepta-color type="text" color="yellow">**Version change**</hepta-color><hepta-color type="text" color="yellow">：</hepta-color><hepta-color type="text" color="yellow">`appVersion`</hepta-color><hepta-color type="text" color="yellow"> stays </hepta-color><hepta-color type="text" color="yellow">`2.1.0`</hepta-color><hepta-color type="text" color="yellow"> (code did not change), but </hepta-color><hepta-color type="text" color="yellow">`version`</hepta-color><hepta-color type="text" color="yellow"> must go from </hepta-color><hepta-color type="text" color="yellow">`1.0.1`</hepta-color><hepta-color type="text" color="yellow"> to </hepta-color><hepta-color type="text" color="yellow">`1.1.0`</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
61	
62	  - <hepta-color type="text" color="yellow">**Meaning**</hepta-color><hepta-color type="text" color="yellow">: contents did not change; we upgraded the infrastructure logic that deploys them.</hepta-color>
63	
64	  The benefit is that you can **track independently** software version vs config version. On rollback, that helps you tell whether new code broke things, or a new K8s config (wrong Service port) did.
65	
66	+ ## Why call this a blueprint?
67	
68	  In Kubernetes we usually face resources (Pod, Service, Ingress). In Helm's view you face a **structured design drawing**。
69	
70	  That structure brings key advantages:
71	
72	  1. **Predictability**: whether Bitnami's Redis Chart or your own API Chart, open the folder and you will find Manifests in `templates/` and settings in `values.yaml`.
73	
74	  2. **Decoupling**: developers watch `appVersion` to keep code right; operators watch `version` to keep deploy architecture sound. They meet in `Chart.yaml` without stepping on each other.
75	
76	  3. **Automation-friendly**: because the tree is fixed, CI/CD can easily find `Chart.yaml` for version info, or scan `templates/` for linting.
77	
78	  With this blueprint you are no longer blindly editing YAML—you are designing a reusable software package. That mindset supports the deeper work ahead.


## Note 81: Chart Directory Blueprint

- id: `9d3ad48f-8459-4b86-8296-e6ca23d140f4`
- last_edited: 2026-10-11T03:49:16.240Z
- whiteboards: DevOp All, Helm and Helmfile

card "Chart Directory Blueprint" [9d3ad48f-8459-4b86-8296-e6ca23d140f4] 40 lines
--- Databases ---
none
1	# Chart Directory Blueprint
2	
3	Have you ever hunted through folders for a simple microservice deploy, trying to remember which `deployment.yaml` is the latest? Or changed an Image Tag and accidentally edited a parameter you should not touch? After Helm concepts and operations, we get practical:**take apart a Chart from the inside**。
4	
5	If Kubernetes YAML files are scattered part drawings, a Helm Chart is a tidy toolbox. For that toolbox to open correctly anywhere, Helm defines a strict directory and metadata spec. This is not just aesthetics—it tells the Helm engine what is the brain (metadata), what is defaults, and what is the production line (templates).
6	
7	## Standard Helm Chart directory tree
8	
9	When you run `helm create my-chart`, Helm generates a standard tree. That tree is the common language of every Helm user. Understand it and you can read any public Chart in the world.
10	
11	```text
12	my-chart/
13	├── Chart.yaml          # The Chart's brain: name, version, description
14	├── values.yaml         # Default values for the user: tunable parameters
15	├── templates/          # Production line: Go templates for Kubernetes resources
16	│   ├── deployment.yaml
17	│   ├── service.yaml
18	│   ├── _helpers.tpl    # Parts shop: reusable snippets
19	│   └── NOTES.txt       # Post-install sticky note: guidance shown to the user
20	├── charts/             # Extra toolbox: subcharts this Chart depends on
21	└── .helmignore         # Exclude list: files that must not be packed into the Chart
22	```
23	
24	### What each file and directory does
25	
26	- **Chart.yaml**: the Chart's ID and manifest. It tells Helm what kind of package this is, which version, and which application it is for.
27	
28	- **values.yaml**: the file you change most. <hepta-color type="text" color="yellow">It holds default values for every template variable. Think of it as factory settings</hepta-color>. Users can override them at install with an external file, without touching core templates.
29	
30	- <hepta-color type="text" color="yellow">**templates/**</hepta-color>: the heart of the Chart. Kubernetes Manifests with Go Template syntax live here.<hepta-color type="text" color="yellow">On install, Helm reads values from </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow">, fills the templates, and sends standard YAML to Kubernetes.</hepta-color>
31	
32	- **templates/\_helpers.tpl**: notice the underscore (`_`)? Files that start with an underscore are not rendered as Kubernetes resources.<hepta-color type="text" color="yellow">This file usually defines named templates—snippets reused across YAML files (label sets, resource-name logic).</hepta-color>。
33	
34	- **templates/NOTES.txt**: a thoughtful design. After a successful `helm install`, the terminal prints this file's rendered content to tell the user how to reach the new service.
35	
36	- **charts/**: if your app depends on other services (for example the API needs Redis), put the Redis Chart here, or download it here via config.
37	
38	<hepta-mention type="card" id="ce934226-23d4-474a-b0b0-472f64fb9260">Chart.yaml in depth: the metadata brain</hepta-mention>
39	
40


## Note 82: Chart Repository

- id: `df2cb80b-42db-4d25-9364-53ad736a9e9c`
- last_edited: 2026-10-11T03:25:54.585Z
- whiteboards: DevOp All, Helm and Helmfile

card "Chart Repository" [df2cb80b-42db-4d25-9364-53ad736a9e9c] 91 lines
--- Databases ---
none
1	# Chart Repository
2	
3	<hepta-color type="text" color="yellow">When you install Redis or Prometheus on Kubernetes, do you write every Deployment and Service YAML from scratch? In Helm, the answer is usually no. Just as developers pull packages from npm and images from Docker Hub, DevOps engineers fetch proven application packages from </hepta-color><hepta-color type="text" color="yellow">**Chart Repository**</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
4	
5	But where do those Charts live? How do you search, download, or even run a private registry in production? Understanding Chart repositories is the last mile from hand-written YAML to automated package management.
6	
7	## Three habitats for Charts
8	
9	Helm does not force Charts into one place. Depending on development stage and infrastructure, you mainly meet three sources.
10	
11	### 1. Traditional HTTP repository (index mode)
12	
13	This is Helm's classic and still most common form. It is simply an **HTTP server** (can be <hepta-color type="text" color="yellow">GitHub Pages</hepta-color>, S3 bucket, or any static file host) with two core pieces:
14	
15	- <hepta-color type="text" color="yellow">`index.yaml`</hepta-color><hepta-color type="text" color="yellow">: the repository catalog. It lists every Chart name, version, description, and download URL.</hepta-color>
16	
17	- <hepta-color type="text" color="yellow">**`.tgz`**</hepta-color><hepta-color type="text" color="yellow">** archives**</hepta-color><hepta-color type="text" color="yellow">: the actual Chart packages.</hepta-color>
18	
19	When you run `helm install`, Helm first reads the `index.yaml` file, finds your version, then downloads the matching `.tgz` and renders it. This is centralized indexing: simple structure, but with a huge Chart count the `index.yaml` file becomes bloated.
20	
21	### 2. OCI registry (tag mode — the modern trend)
22	
23	<hepta-color type="text" color="yellow">This is </hepta-color><hepta-color type="text" color="yellow">**what the Helm community prefers today**</hepta-color><hepta-color type="text" color="yellow">. OCI (Open Container Initiative) is the container image standard, and Helm can now live in </hepta-color><hepta-color type="text" color="yellow">**Container Registry**</hepta-color><hepta-color type="text" color="yellow"> (AWS ECR, Google Artifact Registry, GitHub Packages, or Harbor).</hepta-color> 
24	
25	This no longer depends on `index.yaml`; it uses the same tag mechanism as Docker images.
26	
27	- **Benefit 1: unified management**. Docker images and Helm Charts can share one registry, one IAM permission model, and one asset lifecycle.
28	
29	- <hepta-color type="text" color="yellow">**Benefit 2: better performance**</hepta-color><hepta-color type="text" color="yellow">. No full index download—fetch precisely by tag (for example </hepta-color><hepta-color type="text" color="yellow">`oci://my-reg/charts/my-app:1.2.0`</hepta-color><hepta-color type="text" color="yellow">).</hepta-color>
30	
31	As a DevOps engineer on AWS, storing private Charts in ECR with OCI is usually safer and less work than running your own HTTP server.
32	
33	### 3. Local path
34	
35	<hepta-color type="text" color="yellow">While developing a Chart you do not pack and upload after every line. You reference a directory on the local filesystem.</hepta-color>
36	
37	```bash
38	helm install my-release ./my-chart-directory
39	```
40	
41	That lets you test template changes immediately.
42	
43	---
44	
45	## Core workflow: day-to-day repository interaction
46	
47	Once you know the habitats, you need the Helm CLI to talk to them. It is a standard four-step flow.
48	
49	### Step 1: `helm repo add` (register a source)
50	
51	<hepta-color type="text" color="yellow">Like adding a bookmark. You tell Helm: here is a useful repository; remember it.</hepta-color>
52	
53	<hepta-color type="text" color="yellow">Using the well-known Bitnami community repo:</hepta-color>
54	
55	```bash
56	helm repo add bitnami https://charts.bitnami.com/bitnami
57	```
58	
59	After this command, Helm records the URL in local config and names it `bitnami`。
60	
61	### Step 2: `helm repo update` (sync the cache)
62	
63	**This is the step beginners forget most often.** <hepta-color type="text" color="yellow">Why update? For performance, Helm caches each repository's </hepta-color><hepta-color type="text" color="yellow">`index.yaml`</hepta-color><hepta-color type="text" color="yellow"> on your machine. If Bitnami shipped a new Redis Chart yesterday and you skipped update, your cache still has the old catalog and you will not find the latest version.</hepta-color>
64	
65	> **Pro tip:** In CI/CD we usually run `helm install/upgrade` before `helm repo update` so we fetch the latest Chart info.
66	
67	### Step 3: `helm search repo` (search)
68	
69	When you do not know the Chart name, or want to see available versions:
70	
71	```bash
72	helm search repo redis
73	```
74	
75	This lists Charts containing "redis" across every repository you added. To see all historical versions of a Chart, add the `-l` (versions) flag.
76	
77	### Step 4: `helm pull` (download and inspect)
78	
79	Sometimes you do not want to install the official Chart—you want to learn from its templates or fork it. That is when `helm pull` helps.
80	
81	```bash
82	helm pull bitnami/redis --untar
83	```
84	
85	`--untar` is important: it unpacks automatically so you can see all source under the `templates/` directory.
86	
87	<hepta-color type="text" color="yellow">In production, for stability, DevOps engineers often </hepta-color><hepta-color type="text" color="yellow">`pull`</hepta-color><hepta-color type="text" color="yellow"> an official Chart locally, security-review it or add company labels, then repack it into a private repository.</hepta-color>
88	
89	---
90	
91


## Note 83: Chart, Values, Release

- id: `79fc1ac6-4818-416d-a7c5-9543f6c2dc55`
- last_edited: 2026-10-11T05:38:38.395Z
- whiteboards: DevOp All, Helm and Helmfile

card "Chart, Values, Release" [79fc1ac6-4818-416d-a7c5-9543f6c2dc55] 113 lines
--- Databases ---
none
1	# Chart, Values, Release
2	
3	## The formula: templates plus parameters
4	
5	Helm in one line:
6	
7	**Chart (template) + Values (params) = Kubernetes YAML (render) → Release (instance)**
8	
9	You do not write a Deployment.yaml per environment. You <hepta-color type="text" color="yellow">write one generic template (Chart), </hepta-color><hepta-color type="text" color="yellow">**give each environment a config (Values). Helm renders what the cluster gets and records the install (Release).**</hepta-color>
10	
11	A kitchen analogy for the three ideas.
12	
13	## Chart: the recipe — what to install
14	
15	**Chart** is Helm's package. It describes every Kubernetes resource the app needs.
16	
17	In the analogy, **Chart is the recipe**。  It says:
18	
19	- Which tools (Deployment, Service, Ingress)
20	
21	- The steps (YAML structure and logic)
22	
23	- Default seasoning (resource limits, replica count)
24	
25	A typical Chart has a `templates/` directory of YAML with slots. In `deployment.yaml` you do not hardcode `replicas: 3`; you write `replicas: {{ .Values.replicaCount }}`. The recipe does not say "three tomatoes"; it says "N tomatoes." N is filled later.
26	
27	<hepta-color type="text" color="yellow">The Chart also has an ID — </hepta-color><hepta-color type="text" color="yellow">`Chart.yaml`</hepta-color><hepta-color type="text" color="yellow"> — the recipe version. </hepta-color><hepta-color type="text" color="yellow">`version: 1.2.0`</hepta-color><hepta-color type="text" color="yellow"> is the recipe itself (maybe a new method). </hepta-color><hepta-color type="text" color="yellow">`appVersion: "3.0.0"`</hepta-color><hepta-color type="text" color="yellow"> is the app in the recipe (Nginx, your API).</hepta-color>
28	
29	## Values: the ingredient list — how to configure
30	
31	Recipe plus **ingredient list (Values)** — how this cook goes.
32	
33	<hepta-color type="text" color="yellow">**Values**</hepta-color><hepta-color type="text" color="yellow"> is a set of params that override Chart defaults. Dynamic, so one Chart fits many environments.</hepta-color>
34	
35	- **dev**: 1 replica, 128Mi memory.
36	
37	- **prod**: 3 replicas, 1Gi memory, Ingress on.
38	
39	<hepta-color type="text" color="yellow">When you run </hepta-color><hepta-color type="text" color="yellow">`helm install`</hepta-color><hepta-color type="text" color="yellow"> you tell Helm: this recipe (Chart), this list (Values). Helm fills the slots and renders YAML.</hepta-color>
40	
41	## Release: the dish on the table
42	
43	The cook (Helm) follows recipe and list and serves. That **served dish is the Release**。
44	
45	**Release** is one running instance of a Chart in the cluster. Important:
46	
47	- **Chart is a static asset**: like a git repo or Docker image, on disk or in a repo.
48	
49	- <hepta-color type="text" color="yellow">**Release is dynamic state**</hepta-color><hepta-color type="text" color="yellow">: the set of resources already in the cluster.</hepta-color>
50	
51	The same Chart can install many times into many Releases. One tomato-pasta recipe, two shops — Taipei and Taichung — two plates (`my-app-dev` and `my-app-prod`). Independent in the cluster.
52	
53	Each Release keeps revision history. Too salty (a bad upgrade)? `helm rollback` rolls that Release back.
54	
55	## How the three meet
56	
57	The flow from static work to cluster state:
58	
59	![](heptabase://card/79fc1ac6-4818-416d-a7c5-9543f6c2dc55/file/080bd8d4-fcda-48ec-8381-d26d1a22cfa8)
60	
61	> *Chart as template, plus Values, Helm renders Kubernetes resources and a Release.*
62	
63	### Static vs dynamic duties
64	
65	Once you see the split, ops questions locate faster:
66	
67	| Idea | Analogy | Duty | How often it changes | Where it lives |
68	| --- | --- | --- | --- | --- |
69	| **Chart** | Recipe | Resource structure and logic | <hepta-color type="text" color="yellow">Low (when features change)</hepta-color> | Git / Artifact Hub |
70	| **Values** | Ingredient list | Inject env-specific vars | <hepta-color type="text" color="yellow">Medium (config changes)</hepta-color> | Git / CI secrets |
71	| **Release** | The cooked dish | Live cluster state | <hepta-color type="text" color="yellow">Keeps evolving after deploy</hepta-color> | <hepta-color type="text" color="yellow">K8s cluster (Secret)</hepta-color> |
72	
73	## Multi-env: one recipe, many tastes
74	
75	Handwritten YAML: `dev` wants Nginx `1.21`, `prod` wants `1.22` — two files to edit.
76	
77	In Helm, one **Chart**:
78	
79	```yaml
80	# templates/deployment.yaml (excerpt)
81	spec:
82	  template:
83	    spec:
84	      containers:
85	        - name: nginx
86	          image: "nginx:{{ .Values.imageTag }}"
87	```
88	
89	<hepta-color type="text" color="yellow">Then two </hepta-color><hepta-color type="text" color="yellow">**Values**</hepta-color><hepta-color type="text" color="yellow">：</hepta-color>
90	
91	- <hepta-color type="text" color="yellow">`values-dev.yaml`</hepta-color><hepta-color type="text" color="yellow">: </hepta-color><hepta-color type="text" color="yellow">`imageTag: 1.21`</hepta-color>
92	
93	- <hepta-color type="text" color="yellow">`values-prod.yaml`</hepta-color><hepta-color type="text" color="yellow">: </hepta-color><hepta-color type="text" color="yellow">`imageTag: 1.22`</hepta-color>
94	
95	Install:
96	
97	- `helm install my-app-dev ./chart -f values-dev.yaml` → produces **Release: my-app-dev**
98	
99	- `helm install my-app-prod ./chart -f values-prod.yaml` → produces **Release: my-app-prod**
100	
101	Separation of concerns is why Helm still wins at scale. You manage rules (Chart) and diffs (Values), not duplicated YAML results.
102	
103	## Release lifecycle
104	
105	<hepta-color type="text" color="yellow">A </hepta-color><hepta-color type="text" color="yellow">**Release**</hepta-color><hepta-color type="text" color="yellow"> has a life. First install is Revision 1. Change Values and </hepta-color><hepta-color type="text" color="yellow">`helm upgrade`</hepta-color><hepta-color type="text" color="yellow"> → Revision 2.</hepta-color>
106	
107	Helm does not only render YAML. It manages state. Revision 3 Pods fail? Roll back to 2. Helm reads history from cluster Secrets and restores the Release.
108	
109	## From idea to action
110	
111	Chart, Values, Release is enough theory to operate Helm. Every feature orbits them: find Charts, override Values, manage Release history.
112	
113	Next is Helm CLI. From basic `install` through `upgrade`、`rollback` and dry-run `template` so you can preview a deploy without touching the cluster.


## Note 84: Why Helm Exists

- id: `ccf73782-3693-4e62-a5da-3c79cb6331bb`
- last_edited: 2026-10-11T02:58:47.963Z
- whiteboards: DevOp All, Helm and Helmfile

card "Why Helm Exists" [ccf73782-3693-4e62-a5da-3c79cb6331bb] 41 lines
--- Databases ---
none
1	# Why Helm Exists
2	
3	If you have shipped a few microservices on Kubernetes, you know the scene: a simple API still needs `deployment.yaml`、`service.yaml`、`ingress.yaml`, maybe `configmap.yaml` and `hpa.yaml`. Fine so far. The nightmare starts when you split Dev, Staging, and Prod.
4	
5	<hepta-color type="text" color="yellow">Each environment has different DB strings, replica counts, and CPU/memory limits. Copy three files and edit fields? Ten microservices, three environments, five resource files each is 150 YAML files. Add a label or a security setting everywhere and can you really keep 150 places in sync?</hepta-color>
6	
7	## From hand management to YAML explosion
8	
9	<hepta-color type="text" color="yellow">Without Helm, engineers face </hepta-color><hepta-color type="text" color="yellow">**YAML explosion**</hepta-color><hepta-color type="text" color="yellow"> and </hepta-color><hepta-color type="text" color="yellow">**configuration drift**</hepta-color><hepta-color type="text" color="yellow">.</hepta-color>
10	
11	<hepta-color type="text" color="yellow">The mess of managing many environments by hand:</hepta-color>
12	
13	- <hepta-color type="text" color="yellow">**High duplication**</hepta-color><hepta-color type="text" color="yellow">: Dev and prod </hepta-color><hepta-color type="text" color="yellow">`deployment.yaml`</hepta-color><hepta-color type="text" color="yellow"> are 90% the same. Only a few parameters differ (such as </hepta-color><hepta-color type="text" color="yellow">`image.tag`</hepta-color><hepta-color type="text" color="yellow"> or </hepta-color><hepta-color type="text" color="yellow">`replicas`</hepta-color><hepta-color type="text" color="yellow">).</hepta-color>
14	
15	- <hepta-color type="text" color="yellow">**Hard to sync**</hepta-color><hepta-color type="text" color="yellow">: a field that worked in dev must be copied to prod by hand. Miss one and the deploy fails.</hepta-color>
16	
17	- <hepta-color type="text" color="yellow">**Weak versioning**</hepta-color><hepta-color type="text" color="yellow">: YAML can live in Git, but you cannot see at a glance which config version prod is running. A failed deploy means hunting a commit and re-running </hepta-color><hepta-color type="text" color="yellow">`kubectl apply`</hepta-color><hepta-color type="text" color="yellow"> — slow and dangerous.</hepta-color>
18	
19	### Compare: hand management vs Helm
20	
21	As apps grow, maintenance complexity diverges:
22	
23	| Dimension | Hand management (pure K8s YAML) | With Helm |
24	| --- | --- | --- |
25	| **Multi-environment** | A full YAML set per environment. File count grows linearly. | One template (Chart) plus many parameter files (Values). |
26	| **Parameter changes** | Search-and-replace across files. Easy to miss. | Edit the matching `values.yaml`. |
27	| **Deploy operations** | Many `kubectl apply -f ...` commands. | One `helm install` or `upgrade`。 |
28	| **Rollback** | Find old YAML and apply again by hand. | Use `helm rollback` and revert in a second. |
29	| **Dependencies** | Manually ensure databases and other prerequisites are installed. | Declare them in `Chart.yaml` and download automatically. |
30	
31	## <hepta-color type="text" color="yellow">Helm's role: Kubernetes package manager</hepta-color>
32	
33	<hepta-color type="text" color="yellow">Helm exists to fix that pain. It is the </hepta-color><hepta-color type="text" color="yellow">**Kubernetes package manager**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
34	
35	<hepta-color type="text" color="yellow">Helm packs related resources (Deployment, Service, Ingress...) into a unit called a </hepta-color><hepta-color type="text" color="yellow">**Chart**</hepta-color><hepta-color type="text" color="yellow">. That unit is:</hepta-color>
36	
37	1. <hepta-color type="text" color="yellow">**Versioned**</hepta-color><hepta-color type="text" color="yellow">: give deploy config a version (v1.0.1).</hepta-color>
38	
39	2. <hepta-color type="text" color="yellow">**Reusable**</hepta-color><hepta-color type="text" color="yellow">: one Chart, different parameters, Dev/Staging/Prod.</hepta-color>
40	
41	3. <hepta-color type="text" color="yellow">**Configurable**</hepta-color><hepta-color type="text" color="yellow">: </hepta-color><hepta-color type="text" color="yellow">`values.yaml`</hepta-color><hepta-color type="text" color="yellow"> separates config from templates. Users only change the parameters they care about</hepta-color>


## Note 85: Short Note and QA

- id: `2bbb2464-aa7d-428a-bb32-931a71c91a7b`
- last_edited: 2026-10-11T02:41:23.131Z
- whiteboards: DevOp All, Helm and Helmfile

card "Short Note and QA" [2bbb2464-aa7d-428a-bb32-931a71c91a7b] 605 lines
(Content has 605 lines. Use offset and limit parameters to read a specific portion.)


## Note 86: Certificates and the CA Chain of Trust

- id: `8e7eea12-cdf4-406b-8f0f-e45bc022fd3e`
- last_edited: 2026-10-11T02:22:50.975Z
- whiteboards: DevOp All, Computer Network Foundation

card "Certificates and the CA Chain of Trust" [8e7eea12-cdf4-406b-8f0f-e45bc022fd3e] 102 lines
--- Databases ---
none
1	# Certificates and the CA Chain of Trust
2	
3	Last section we saw HTTP's three fatal wounds: eavesdropping, tampering, and impersonation. HTTPS uses TLS (Transport Layer Security) to fix them. But there is a logic clash: if you are about to encrypt a connection to a server that claims to be Google, how do you know it is Google, not an attacker in the middle?
4	
5	If we cannot confirm identity, even encrypted talk may just be a private chat with a fraud. That is where **digital certificates** and the **CA chain of trust** shine.
6	
7	## Digital certificates: a server's ID card
8	
9	In real life, a bank teller asks for your ID. It has your name, photo, and ID number, stamped by the government. The bank trusts the government, the government attests you are you, so the bank serves you.
10	
11	On the internet, a digital certificate plays the same role. When your browser connects to `https://google.com`, the first thing the server does is toss a certificate: "Hey, this is my ID. It proves I am Google."
12	
13	### X.509 certificate structure
14	
15	The internet's common certificate standard is **X.509**. When you click the lock next to the URL and open certificate details, you are reading this standardized document. A typical X.509 certificate has these key fields:
16	
17	1. **Subject**: who is this certificate for? It usually includes a **Common Name (CN)**, the domain (for example `*.google.com`). Modern certificates more often use **Subject Alternative Name (SAN)** to list many supported domains.
18	
19	2. **Public Key**: the most important part. The certificate's core job is binding identity (domain) to a public key. Later encryption needs that public key to exchange a secret with the server.
20	
21	3. <hepta-color type="text" color="yellow">**Issuer**</hepta-color><hepta-color type="text" color="yellow">: who issued this certificate? That is the </hepta-color><hepta-color type="text" color="yellow">**Certificate Authority (CA)**</hepta-color><hepta-color type="text" color="yellow">, for example DigiCert, GlobalSign, or Let's Encrypt.</hepta-color>
22	
23	4. **Validity**: not-before and not-after dates. For safety, most certificates now last about 90 days or a year.
24	
25	5. **Digital Signature**: the CA encrypts all of the above with its private key. That seal stops the certificate from being tampered with.
26	
27	### Why this paper cannot be forged
28	
29	You may ask: if a certificate is just data, can't an attacker write one that says `Subject: google.com` and send it to me?
30	
31	Yes, but the **digital signature** on a fake certificate will fail. How signatures work:
32	
33	1. <hepta-color type="text" color="yellow">The CA takes all certificate content (domain, public key, validity) and hashes it into a unique fingerprint.</hepta-color>
34	
35	2. <hepta-color type="text" color="yellow">The CA encrypts that fingerprint with its </hepta-color><hepta-color type="text" color="yellow">**private key**</hepta-color><hepta-color type="text" color="yellow">. That is the signature.</hepta-color>
36	
37	3. <hepta-color type="text" color="yellow">The browser uses the CA's </hepta-color><hepta-color type="text" color="yellow">**public key**</hepta-color><hepta-color type="text" color="yellow"> to unlock the signature.</hepta-color>
38	
39	4. <hepta-color type="text" color="yellow">If the unlocked fingerprint matches the hash the browser computes itself, the content was not changed and the CA really issued it.</hepta-color>
40	
41	## Certificate Authority (CA): notary of the digital world
42	
43	If a certificate is an ID card, the **Certificate Authority (CA)** is the government that issues IDs.
44	
45	In HTTPS, the CA is a trusted third party. To keep that status, CAs follow strict security and audit rules. When a company (Google) asks a CA for a certificate, the CA does **validation**。
46	
47	Validation strictness comes in levels:
48	
49	- **Domain Validation (DV)**: most common. The CA only checks you own the domain (often by asking you to add a DNS record). Let's Encrypt automated issuance is this kind.
50	
51	- **Organization Validation (OV)**: besides the domain, legal documents prove the company exists.
52	
53	- **Extended Validation (EV)**: the strictest review. Browsers used to show a green company name; most have dropped that visual difference.
54	
55	## Chain of Trust: how trust is passed
56	
57	Another logic problem: how does the browser know the CA's public key is real? Do we visit DigiCert's office with a USB stick?
58	
59	To make this work at scale we use a **chain of trust**. A relay of trust:
60	
61	### 1. Root CA certificate
62	
63	The start of trust. A root certificate is self-signed by the CA. That sounds odd, but at the end of the chain something must be trusted because it is itself. There are only a few hundred globally accepted roots.
64	
65	### 2. Intermediate CA certificate
66	
67	Root CAs usually do not issue certificates to ordinary sites. Why? The root private key is too important. If it leaks, internet security collapses. So the root key stays in a physically isolated vault and is used only to sign a few intermediate CA certificates.
68	Those intermediate CAs are authorized agents that handle daily high-volume site certificates.
69	
70	### 3. Leaf / end-entity certificate
71	
72	This is the certificate your site (for example `google.com`) receives. It sits at the end of the chain, so it is called a leaf certificate.
73	
74	### Verification: walk upward
75	
76	When the browser gets the certificate for `google.com`, it plays find-the-parent:
77	
78	1. **Leaf** says: I am `google.com`, signed by **Intermediate CA A**.
79	
80	2. Browser checks: do I have **Intermediate CA A**'s public key? Yes, that checks out. But who proves this intermediate CA is reliable?
81	
82	3. **Intermediate CA A** says: I was signed by **Root CA X**.
83	
84	4. Browser checks: do I have **Root CA X**'s public key? Yes, and **Root CA X** is in my trust list!
85	
86	5. **Trust achieved**。
87	
88	## <hepta-color type="text" color="yellow">The browser Root Store</hepta-color>
89	
90	<hepta-color type="text" color="yellow">So where does trust actually start?</hepta-color>
91	
92	<hepta-color type="text" color="yellow">In your OS (Windows, macOS, Linux) or browser (Chrome, Firefox). When you install them, they already include a </hepta-color><hepta-color type="text" color="yellow">**Root Store**</hepta-color><hepta-color type="text" color="yellow">: a list of public keys for hundreds of authoritative CA roots worldwide.</hepta-color>
93	
94	- <hepta-color type="text" color="yellow">**Microsoft**</hepta-color><hepta-color type="text" color="yellow"> has its own root program.</hepta-color>
95	
96	- <hepta-color type="text" color="yellow">**Apple**</hepta-color><hepta-color type="text" color="yellow"> manages macOS and iOS roots.</hepta-color>
97	
98	- <hepta-color type="text" color="yellow">**Mozilla (Firefox)**</hepta-color><hepta-color type="text" color="yellow"> maintains a well-known open-source root list.</hepta-color>
99	
100	- <hepta-color type="text" color="yellow">**Google (Chrome)**</hepta-color><hepta-color type="text" color="yellow"> used to rely on the OS, and now also pushes its own Chrome Root Store.</hepta-color>
101	
102	<hepta-color type="text" color="yellow">That is why you do not need to know every server. You trust your browser vendor (Apple or Google). They pick the most trustworthy CAs and preload those root certificates on your device.</hepta-color><hepta-color type="text" color="yellow">**Trust flows downward from there.**</hepta-color>


## Note 87: Why You Need HTTPS

- id: `08da616b-c9bf-44f2-b03d-b962ef4a27ae`
- last_edited: 2026-10-11T02:04:37.008Z
- whiteboards: DevOp All, Computer Network Foundation

card "Why You Need HTTPS" [08da616b-c9bf-44f2-b03d-b962ef4a27ae] 149 lines
--- Databases ---
none
1	# Why You Need HTTPS
2	
3	After HTTP request/response, methods, and status codes, it can feel like you own internet communication. Look back and a huge hole remains: in HTTP, every bit of information **plaintext** runs naked on the wire.
4	
5	Like writing your bank password on a postcard and handing it to a stranger to take to the bank. Before it arrives it passes postal workers, sorters, drivers, even a neighbor. Anyone who looks down can read it.
6	
7	In modern networks, that plaintext nature leaves HTTP almost undefended. This part takes the attacker's view: HTTP's three security weaknesses, and why we must upgrade HTTP to HTTPS.
8	
9	## HTTP in essence: a postcard in transit
10	
11	Why do we call HTTP plaintext?
12	
13	<hepta-color type="text" color="yellow">When the browser sends an HTTP request (e.g. </hepta-color><hepta-color type="text" color="yellow">`POST /login`</hepta-color><hepta-color type="text" color="yellow">), the packet leaves your laptop, hits the home router, the ISP core, undersea cables, the target DC border router, then the server.</hepta-color>
14	
15	Every hop can read or even change the packet. What you see in DevTools, those hops can see too.
16	
17	The threat model HTTP ignores splits into three questions:
18	
19	1. **Eavesdropping**: Can others see what I am saying?
20	
21	2. **Tampering**: Can others quietly change what I said?
22	
23	3. **Impersonation**: Is the other party who they claim to be?
24	
25	We take these one by one.
26	
27	## Threat 1: Eavesdropping
28	
29	Eavesdropping is the most basic, hardest-to-notice attack. Goal: steal your privacy while you never know.
30	
31	### Where does it happen?
32	
33	You sit in a cafe on free Wi-Fi. You log into an old HTTP-only forum or internal admin.
34	
35	When you hit Login, account and password leave in an HTTP body. Because it is plaintext, anyone on that Wi-Fi (or a dishonest barista) with a simple **packet sniffer (e.g. Wireshark)** can pull your password out of the air.
36	
37	### Why this matters for DevOps
38	
39	You might think: I never do sensitive work on public Wi-Fi. Eavesdropping is not only cafes.
40	
41	In enterprise networks, if Kubernetes service-to-service traffic is plain HTTP, anyone who reaches that network layer (e.g. a hacker with one Pod) can listen to unencrypted traffic on the node or cluster. That is why Zero Trust encrypts even inside the network.
42	
43	**Eavesdropping breaks confidentiality.**
44	
45	## Threat 2: Tampering
46	
47	If eavesdropping is peeking, tampering is meddling. Classic **man-in-the-middle (MITM)**。
48	
49	### Attack scene: ad injection and malware
50	
51	Ever browse a plain HTTP site and a jarring overlay ad pops in, or the URL is right but the page looks wrong?
52	
53	A hop (a shady small ISP or a compromised public gateway) may be at work. As the HTTP response passes, the attacker can:
54	
55	1. **Intercept**: Catch it before it reaches you.
56	
57	2. **Modify**: Inject malicious JavaScript into the HTML, or change a transfer account to theirs.
58	
59	3. **Repack**: Send the altered packet as if nothing happened.
60	
61	### A grim predict-and-reveal
62	
63	**Predict**: You download an installer from an official site (e.g. a `.exe` or `.dmg` file) over HTTP only. What can the attacker do?
64	
65	**Reveal**: They can swap the binary in the download stream for a trojan. The browser says done, the filename is right, but you run the attacker's program. HTTP has no way to check that what you got matches what the server sent.
66	
67	**Tampering breaks integrity.**
68	
69	## Threat 3: Impersonation
70	
71	The deadliest. Even if the path is safe and no one peeks or edits — what if the other party is fake?
72	
73	### Who is the real bank?
74	
75	You try to reach `mybank.com`. An attacker uses **DNS spoofing** or other redirects so the browser thinks the attacker's server (IP `1.2.3.4`) is the real bank.
76	
77	On plain HTTP the browser happily opens TCP to `1.2.3.4` and sends HTTP. The attacker serves a login page that looks like the bank. HTTP cannot prove identity, so you cannot tell if "enter your password" is the bank or a crook.
78	
79	### The cost of impersonation
80	
81	This is an advanced form of **phishing**. If identity cannot be verified, encryption is pointless — you may be encrypting your password to the hacker.
82	
83	**Impersonation breaks authenticity.**
84	
85	## How TLS treats the disease
86	
87	To see how HTTPS (HTTP over TLS/SSL) solves this, use a security model: the **CIA triad**. HTTPS is not magic. It inserts a **TLS（Transport Layer Security）** layer between HTTP and TCP, with a fix for each of the three problems.
88	
89	### 1. Against eavesdropping: encryption
90	
91	<hepta-color type="text" color="yellow">TLS uses strong crypto to turn plaintext into </hepta-color><hepta-color type="text" color="yellow">**ciphertext**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
92	
93	- <hepta-color type="text" color="yellow">**Effect**</hepta-color><hepta-color type="text" color="yellow">: Even if the attacker intercepts the packet, they see garbage (e.g. </hepta-color><hepta-color type="text" color="yellow">`x86!@#$90...`</hepta-color><hepta-color type="text" color="yellow">) and cannot read it without the key.</hepta-color>
94	
95	- **Goal**: Confidentiality.
96	
97	### 2. Against tampering: message integrity
98	
99	<hepta-color type="text" color="yellow">TLS attaches a digital fingerprint to every packet (usually HMAC).</hepta-color>
100	
101	- <hepta-color type="text" color="yellow">**Effect**</hepta-color><hepta-color type="text" color="yellow">: If a MITM flips even one bit, the receiver's fingerprint check fails and the packet is dropped. Tampering cannot hide.</hepta-color>
102	
103	- **Goal**: Integrity.
104	
105	### 3. Against impersonation: certificates
106	
107	This is HTTPS's core. The server must show the browser a **digital certificate**。
108	
109	- **Effect**: The cert is the server's ID card. The browser checks it is not forged, not expired, and issued by a trusted CA.
110	
111	- **Goal**: Authentication.
112	
113	---
114	
115	## Why this matters for Kubernetes and DevOps
116	
117	As a DevOps engineer, this threat model helps you configure systems correctly.
118	
119	When you expose an app through Ingress, you typically configure **TLS termination**. What does this actually mean?
120	
121	- **Why "Termination"?** The encrypted TLS session from the browser ends right at the Ingress controller. Ingress decrypts the traffic and forwards plain HTTP to backend Pods inside your private cluster network.
122	
123	- **Why do it here?** It provides HTTPS encryption over the vulnerable public internet without burdening every individual Pod with certificate management and decryption overhead.
124	
125	- Without setting up TLS on Ingress, user traffic on the public net travels in plaintext, exposing you to all three threats above.
126	
127	- Modern browsers (Chrome/Edge) mark non-HTTPS sites "Not secure", which hurts trust.
128	
129	- Under strict compliance (e.g. PCI DSS), skipping HTTPS can be illegal.
130	
131	After why, we need how. HTTPS succeeds only if we can answer: **why should I trust this certificate?** That is the next part.
132	
133	### Recap
134	
135	- **HTTP is plaintext**, like sending a postcard.
136	
137	- **Eavesdropping** threatens **confidentiality** and leaks passwords.
138	
139	- **Tampering** threatens **integrity** and lets attackers inject malware.
140	
141	- **Impersonation** threatens **authenticity** and connects you to the wrong party.
142	
143	- **HTTPS = HTTP + TLS** solves all three with encryption, signatures, and certificates.
144	
145	## Certificates and where trust starts
146	
147	TLS uses certificates against impersonation. A logic hole: what if the hacker issues their own "I am Google" cert? Why should the browser trust the paper the server holds up?
148	
149	Next we cover **X.509 certificate structure** and how the internet built a **CA trust chain** so you in Taiwan can safely trust a server in the US.


## Note 88: HTTP/1.1 vs HTTP/2

- id: `50f9f650-a53b-4876-834b-a6524afdb9aa`
- last_edited: 2026-10-11T01:56:48.449Z
- whiteboards: DevOp All, Computer Network Foundation

card "HTTP/1.1 vs HTTP/2" [50f9f650-a53b-4876-834b-a6524afdb9aa] 120 lines
--- Databases ---
none
1	# HTTP/1.1 vs HTTP/2
2	
3	If you watched early page loads, images appeared one after another. A large image stuck, and later icons never started. That queue is not an illusion. It is how HTTP/1.1 was designed.
4	
5	Picture a supermarket with one checkout. The cart ahead of you is huge, and they are arguing a discount. You only have a bottle of water, and you wait. That is head-of-line blocking. HTTP/2 turns that one-counter shop into a modern warehouse with conveyor belts that handle hundreds of items at once.
6	
7	## HTTP/1.1’s bind: readable, inefficient
8	
9	Before HTTP/2, why did twenty-year-old HTTP/1.1 suddenly feel not good enough?
10	
11	As we saw last section, <hepta-color type="text" color="yellow">HTTP/1.1 is a </hepta-color><hepta-color type="text" color="yellow">**plain-text**</hepta-color><hepta-color type="text" color="yellow"> protocol.</hepta-color> In the 1990s that was perfect — humans could even type `telnet` by hand in `GET /index.html HTTP/1.1` to simulate a browser. That <hepta-color type="text" color="yellow">readability became a heavy burden on the modern web.</hepta-color>
12	
13	### 1. Application-layer head-of-line blocking (Layer 7 HOL)
14	
15	HTTP/1.1 is request–response on one TCP connection: you must finish receiving the previous response before you send the next request.
16	
17	That is a bottleneck. If HTML needs 50 CSS, JS, and image files, and the first JS is slow or huge, the other 49 sit in the send queue. One slow car on a one-lane road blocks every sports car behind it.
18	
19	### 2. Browser connection limits and workarounds
20	
21	To ease that block, modern browsers (Chrome, Firefox) open up to **6 TCP connections**。
22	
23	Six counters instead of one helps, at a new cost:
24	
25	- **Handshake tax**: every new TCP connection needs a three-way handshake (SYN, SYN-ACK, ACK). HTTPS adds a TLS handshake. Time and resources add up.
26	
27	- **Slow start**: recall TCP congestion control — a new connection starts slow. Those 6 connections cannot use full bandwidth at first.
28	
29	To dodge the 6-connection cap, early frontend engineers invented tricks:
30	
31	- **Domain Sharding**: put images on `img1.example.com`, scripts on `static.example.com`. The browser opens 6 connections per hostname and beats the limit. The network architecture becomes messy and hard to run.
32	
33	- **Image Sprites**: stitch hundreds of icons into one huge image, download once, slice with CSS.
34	
35	- **Concatenation**: merge dozens of JS files into one multi-MB blob.
36	
37	They work. They are still patches on HTTP/1.1.
38	
39	### 3. Fat, repeated headers (header bloat)
40	
41	HTTP is stateless, so every request carries enough context. In modern web apps, headers got huge.
42	
43	- **User-Agent**: full browser version.
44	
45	- **Cookies**: sometimes several KB.
46	
47	- **Referer**: the referring URL.
48	
49	In HTTP/1.1 those headers are **plain text and uncompressed**. Worse, 50 requests on the same site send almost the same User-Agent and Cookies. You may ship 1000 bytes of headers for a 10-byte icon. On mobile that wastes a lot of bandwidth.
50	
51	---
52	
53	## HTTP/2’s change: binary and multiplexing
54	
55	Google built SPDY in the early 2010s; it became HTTP/2’s prototype. HTTP/2 stopped optimizing for “easy to read” and optimized for efficiency.
56	
57	### Binary framing: a new language underneath
58	
59	This is HTTP/2’s deepest change. HTTP/1.1 deals in lines of text. HTTP/2 deals in **frames**。
60	
61	<hepta-color type="text" color="yellow">HTTP/2 splits every request and response into smaller binary units.</hepta-color>
62	
63	- **HEADERS frames**: HTTP headers only.
64	
65	- **DATA frames**: the message body (payload).
66	
67	Machines parse bits far faster than strings. From handwritten letters to structured binary packets: fewer errors, much faster. Everything else in HTTP/2 sits on this.
68	
69	### Multiplexing: end HOL blocking
70	
71	This is HTTP/2’s big move, and the idea DevOps most needs.
72	
73	In HTTP/2 the browser opens **one TCP connection** to the server. All requests (streams) run in parallel on it.
74	
75	- **Streams**: each request/response pair is a stream with a unique ID.
76	
77	- <hepta-color type="text" color="yellow">**Interleaving**</hepta-color><hepta-color type="text" color="yellow">: 50 resources no longer queue. HTTP/2 can send all 50 at once. </hepta-color><hepta-color type="text" color="yellow">**The server can interleave fragments of image A, script B, and stylesheet C on the same TCP pipe.**</hepta-color>
78	
79	When fragments arrive, the browser uses each frame’s **Stream ID** and reassembles the files like a puzzle.
80	
81	**What changes?**
82	
83	1. <hepta-color type="text" color="yellow">**Fixes L7 HOL blocking**</hepta-color><hepta-color type="text" color="yellow">: a large, slow image A does not block script B behind it.</hepta-color>
84	
85	2. <hepta-color type="text" color="yellow">**Fewer handshakes**</hepta-color><hepta-color type="text" color="yellow">: one TCP connection is enough. No more constant SYN/ACK.</hepta-color>
86	
87	3. **Better TCP use**: that one connection leaves slow start quickly and reaches full throughput.
88	
89	### Header compression (HPACK): kill repeats
90	
91	If headers repeat on the same connection, why not remember them?
92	
93	HTTP/2 uses the **HPACK algorithm**. The idea:
94	
95	1. **Static table**: both sides pre-agree common headers (e.g. `GET` is 2, `POST` is 3, `authority` is 1). On the wire you send a number.
96	
97	2. **Dynamic table**: a shared cache. After the first request sends a long `User-Agent`, the server remembers it. The second request says “same header ID as before,” and the server reconstructs it.
98	
99	Headers shrink 80–90%. On mobile that means less delay and less data.
100	
101	### Server Push
102	
103	In HTTP/1.1 the browser must finish parsing HTML, see `main.css`, then request CSS.
104	HTTP/2 lets the server guess. When the browser asks for `index.html`, the server can push `main.css` into the browser cache. When the parser needs CSS, it is already there.
105	
106	*Note: Server Push sounds strong, but cache sync and complexity limited adoption. Chrome even plans to drop it. Still, it was HTTP/2’s serious try at cutting latency.*
107	
108	---
109	
110	## DevOps and Kubernetes: why you should care
111	
112	We often say protocols are for people. In cloud-native, protocols are for microservices.
113	
114	### 1. gRPC and HTTP/2
115	
116	If you use **gRPC** for service-to-service calls in Kubernetes, you are deep on HTTP/2. gRPC requires HTTP/2 underneath, mainly because:
117	
118	- **Low latency**: microservices make huge numbers of RPCs. HTTP/2 multiplexing runs thousands of requests on one connection.
119	
120	- **Streaming**: HTTP/2 supports long-lived connections and bidirectional streams — live metrics, long jobs.


## Note 89: HTTP Headers

- id: `602b0846-c1f4-465b-a655-3d8035b8e49b`
- last_edited: 2026-10-11T01:50:27.887Z
- whiteboards: DevOp All, Computer Network Foundation

card "HTTP Headers" [602b0846-c1f4-465b-a655-3d8035b8e49b] 160 lines
--- Databases ---
none
1	# HTTP Headers
2	
3	Imagine mailing a package. The contents are the **Body** (body). The action on the wrapper ("sign" or "return") is the **Method** (method). What are **HTTP headers**?
4	
5	They are the **labels and notes** on the box: fragile, keep cold, sender is VIP, contents are PDF, deliver by 2 p.m. tomorrow.
6	
7	Last section: **Methods** (GET, POST, …) express intent; **Status Codes** explain the result. Action plus result is not enough. <hepta-color type="text" color="yellow">On POST, is the body JSON or an image? Where do credentials go for a protected resource? Ten sites on one IP — which one did you want?</hepta-color>
8	
9	<hepta-color type="text" color="yellow">The answers live in HTTP headers — this metadata.</hepta-color>
10	
11	## What headers are: conversation metadata
12	
13	In CS, **metadata**<hepta-color type="text" color="yellow">**（Metadata）**</hepta-color><hepta-color type="text" color="yellow"> is data about data</hepta-color>. <hepta-color type="text" color="yellow">HTTP headers are classic metadata. They are not the business payload (that is Body). They define the conversation's </hepta-color><hepta-color type="text" color="yellow">**environment, format, auth, and behavior**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
14	
15	### Header structure
16	
17	<hepta-color type="text" color="yellow">Simple, readable key-value pairs, colon-separated:</hepta-color>
18	
19	`Key: Value`
20	
21	For example:
22	
23	- `Content-Type: application/json`
24	
25	- `Host: api.example.com`
26	
27	**Technical details:**
28	
29	- **Case-insensitive**: keys usually ignore case (`Content-Type` and `content-type` are the same). In practice we capitalize for readability.
30	
31	- **Multi-value**: some headers take several values, comma-separated.
32	
33	- **Two classes**：
34	
35	  1. **Request headers**: client tells the server who it is, which format it wants, credentials.
36	
37	  2. **Response headers**: server tells the client who it is, the body's format, how long to cache.
38	
39	## Host: soul of the modern web and Kubernetes
40	
41	If a DevOps engineer picks one header among hundreds, it is \*\*`Host`\*\*。
42	
43	### Why Host exists
44	
45	Early on, one IP meant one physical server and one site. One family in a building: know the address (IP), find the person.
46	
47	With cloud and virtualization: **one IP may host thousands of sites**. Virtual hosting. An office tower (same IP) with Google, Amazon, Netflix. TCP is up at the door; without a company name the desk (server) cannot route you.
48	
49	**`Host`**** header is the company name you tell the desk.**
50	
51	### Host in the request flow
52	
53	You type `https://tomtom-app.com`:
54	
55	1. Browser DNS lookup → IP `1.2.3.4`。
56	
57	2. Browser opens TCP to `1.2.3.4` port 443.
58	
59	3. Then the HTTP request. **This is the key**: even though the connection is to `1.2.3.4`, the browser must still send `Host: tomtom-app.com`。
60	
61	The server reads the `Host` header: you want `tomtom-app.com`, not `another-app.com` on the same box — then routes to the right logic.
62	
63	### Kubernetes practice: how Ingress works
64	
65	In Kubernetes this is how **Ingress Controller** (e.g. Nginx Ingress) works.
66	
67	A cluster usually has one public LoadBalancer IP. You may have ten `Service` resources for ten domains, all pointing at that Ingress Controller IP.
68	
69	An Ingress resource:
70	
71	```yaml
72	apiVersion: networking.k8s.io/v1
73	kind: Ingress
74	metadata:
75	  name: my-app-ingress
76	spec:
77	  rules:
78	  - host: tomtom-app.com  # <--- Ingress Controller matches this!
79	    http:
80	      paths:
81	      - path: /
82	        backend:
83	          service:
84	            name: tomtom-service
85	            port:
86	              number: 80
87	```
88	
89	The Ingress Controller watches rules and keeps a map. Incoming traffic: it inspects the **`Host`**** header**. If it is `tomtom-app.com`, it forwards to `tomtom-service`; if `Host` matches no rule, usually **404 Not Found** (or a default backend).
90	
91	**DevOps pain warning**:
92	Why `curl <IP_ADDRESS>` against a K8s service often 404s: `curl` defaults Host to the IP as the `Host` header, and Ingress has no rule for that IP.
93	**Fix**: use `curl -H "Host: tomtom-app.com" <IP_ADDRESS>` to send the right header.
94	
95	## Content negotiation: Content-Type and Accept
96	
97	This pair is the data's language and format — how client and server understand each other.
98	
99	### Content-Type: what am I sending?
100	
101	**`Content-Type`** tells the receiver how to parse the Body.
102	
103	- JSON: header is `Content-Type: application/json`。
104	
105	- PNG upload: header is `Content-Type: image/png`。
106	
107	- HTML form: header may be `application/x-www-form-urlencoded`。
108	
109	Wrong header? Server sends image bytes labeled `text/plain`. The browser opens it as text: garbage.
110	
111	### Accept: what do I want?
112	
113	**`Accept`** is a request header: the format the client wants.
114	A well-designed API can branch on it:
115	
116	- Client A sends `Accept: application/json`; server returns JSON.
117	
118	- Client B sends `Accept: application/xml`; the same API may return XML.
119	
120	"I want pasta (Accept)" vs "here is your pasta (Content-Type)" is **content negotiation**。
121	
122	## Identity and security: Authorization and User-Agent
123	
124	### Authorization: ticket at the gate
125	
126	<hepta-color type="text" color="yellow">Modern microservices rarely use classic cookie login. They use tokens. The </hepta-color><hepta-color type="text" color="yellow">**`Authorization`**</hepta-color><hepta-color type="text" color="yellow"> header carries those credentials.</hepta-color>
127	
128	<hepta-color type="text" color="yellow">Most common is </hepta-color><hepta-color type="text" color="yellow">**Bearer Token**</hepta-color><hepta-color type="text" color="yellow"> (often JWT):
129	</hepta-color><hepta-color type="text" color="yellow">`Authorization: Bearer <your_jwt_token>`</hepta-color>
130	
131	At the server or API Gateway, extract the header, check token validity and expiry, then allow or deny.
132	
133	**DevOps tip**:
134	Debugging `401 Unauthorized`: first check whether `Authorization` is missing, or a space after Bearer.
135	
136	### User-Agent: who are you?
137	
138	**`User-Agent`** describes the client software, e.g.:
139	
140	- `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)...` (Chrome)
141	
142	- `curl/7.64.1` (curl)
143	
144	- `PostmanRuntime/7.26.8` (Postman)
145	
146	Useful for mobile vs desktop content, log traffic sources, and WAF blocking bot User-Agents.
147	
148	## Performance: Cache-Control 
149	
150	Advanced HTTP, essential for high-traffic ops.
151	
152	### Cache-Control: cache instructions
153	
154	**`Cache-Control`** response header tells the browser or CDN (e.g. Cloudflare) how long to cache.
155	
156	- `Cache-Control: max-age=3600`: valid for one hour (3600s); do not revalidate; use your cache.
157	
158	- `Cache-Control: no-store`: sensitive (e.g. bank balance); never cache.
159	
160	Huge for performance. Correct cache headers on CSS/JS can cut server load 80%+.


## Note 90: Methods and Status Codes

- id: `f2971263-21d4-44af-8970-23826426a83a`
- last_edited: 2026-10-11T01:46:19.954Z
- whiteboards: DevOp All, Computer Network Foundation

card "Methods and Status Codes" [f2971263-21d4-44af-8970-23826426a83a] 110 lines
--- Databases ---
none
1	# Methods and Status Codes
2	
3	You run an ecommerce site on Kubernetes. The board goes red; error rate spikes. Logs show a mix of `GET /cart 200`, `POST /checkout 504`, and `PUT /profile 403`。
4	
5	Without reading backend code, can methods and status codes tell you if the DB is slow, auth is wrong, or the load balancer is misconfigured?
6	
7	That is HTTP's semantic power. HTTP is not only a pipe; it has a strict grammar. Learn it and DevOps debugging gets an instinct for the real fault.
8	
9	## Request methods: state your intent
10	
11	Last section, the first line of an HTTP request was **Request Line**. The first word is the method. Methods are verbs: what do you want to do to this resource?
12	
13	### The five core verbs
14	
15	<hepta-color type="text" color="yellow">HTTP defines many methods. In modern APIs and DevOps you spend 95% of time on these five:</hepta-color>
16	
17	1. **GET — read a resource**:
18	   Most common. Typing a URL or `kubectl get pods` is GET. Use it only to fetch data; it must not have side effects.
19	
20	2. **POST — **<hepta-color type="text" color="yellow">**Create**</hepta-color>** a resource**:
21	   A signup form, or in K8s `kubectl apply -f deployment.yaml` (first create) usually maps to POST. It creates something new on the server.
22	
23	3. **PUT — **<hepta-color type="text" color="yellow">**Full replace of a resource**</hepta-color>:
24	   A strong verb. Update a user's entire record with a PUT that carries the full new body. The server overwrites the old resource.
25	
26	4. **PATCH — **<hepta-color type="text" color="yellow">**Partial update of a resource**</hepta-color>:
27	   Change only email, leave address and name. PATCH is a patch on specific fields.
28	
29	5. **DELETE — delete a resource**:
30	   Plain meaning: ask the server to delete the named resource.
31	
32	### Safe and Idempotent
33	
34	Knowing the verbs is not enough. These two properties affect system stability.
35	
36	**Predict:** Network lag; you double-click Submit Order. Should the server place two orders, or one?
37	
38	That is **idempotency**.
39	
40	- **Safe**: the method does not change server state.
41	
42	  - <hepta-color type="text" color="yellow">**GET**</hepta-color><hepta-color type="text" color="yellow"> is safe. Read a page a hundred times; data should not change (ignore log counters).</hepta-color>
43	
44	- <hepta-color type="text" color="yellow">**Idempotent**</hepta-color><hepta-color type="text" color="yellow">: one call or many, same result.</hepta-color>
45	
46	  - <hepta-color type="text" color="yellow">**GET**</hepta-color><hepta-color type="text" color="yellow"> is idempotent: read once or ten times, same content (if nobody else changed it).</hepta-color>
47	
48	  - <hepta-color type="text" color="yellow">**DELETE**</hepta-color><hepta-color type="text" color="yellow"> is idempotent: delete once, gone; delete again, still gone (server may 404, resource state does not change further).</hepta-color>
49	
50	  - <hepta-color type="text" color="yellow">**PUT**</hepta-color><hepta-color type="text" color="yellow"> is idempotent: replace A with B ten times, result is still B.</hepta-color>
51	
52	  - <hepta-color type="text" color="yellow">**POST is not idempotent**</hepta-color><hepta-color type="text" color="yellow">: the dangerous case. POST an order twice and you may get two order IDs. On flaky networks, retrying POST needs extra protection (Idempotency Key).</hepta-color>
53	
54	<hepta-color type="text" color="yellow">**Practice:**</hepta-color><hepta-color type="text" color="yellow"> Honor these semantics in APIs and Ingress rules. A GET that deletes DB rows is a disaster: caches and browser prefetch will fire GET on their own.</hepta-color>
55	
56	---
57	
58	## Status codes: the server's answer
59	
60	After handling the request, the server returns a three-digit number on the **Status Line**. That number is a machine-readable diagnosis.
61	
62	### Five classes
63	
64	The first digit is the class:
65	
66	- **1xx (Informational)**: request received, continue. Rare in day-to-day work.
67	
68	- **2xx (Success)**: success, it worked.
69	
70	- **3xx (Redirection)**: the thing moved; here is where to look.
71	
72	- <hepta-color type="text" color="yellow">**4xx (Client Error)**</hepta-color><hepta-color type="text" color="yellow">: you (the client) are wrong. Bad params, no ticket, or the wrong place.</hepta-color>
73	
74	- <hepta-color type="text" color="yellow">**5xx (Server Error)**</hepta-color><hepta-color type="text" color="yellow">: I (the server) failed. Code exception, DB down, or backend timeout.</hepta-color>
75	
76	### DevOps debugging cheat sheet
77	
78	In logs, map codes to meaning immediately:
79	
80	#### 200 OK vs 201 Created
81	
82	- **200** is the generic success code.
83	
84	- **201** is more precise: resource created. Useful in scripts to confirm POST actually produced an object.
85	
86	#### 301 Moved Permanently vs 302 Found (Temporary Redirect)
87	
88	The difference matters a lot for **caching** and **SEO**:
89	
90	- **301**: permanent move. The browser remembers and never asks the old URL again.
91	
92	- **302**: temporary trip. Next time the browser still asks the old URL first.
93	
94	- **Scene**: site migration with 302 and search ranking will not transfer cleanly to the new URL.
95	
96	#### 401 Unauthorized vs 403 Forbidden
97	
98	These two get mixed up constantly:
99	
100	- <hepta-color type="text" color="yellow">**401**</hepta-color><hepta-color type="text" color="yellow">: who are you? (auth failed). No ID, or wrong password.</hepta-color>
101	
102	- <hepta-color type="text" color="yellow">**403**</hepta-color><hepta-color type="text" color="yellow">: I know who you are, but you may not look (authorization failed). Badge is valid; you tried the CFO's office.</hepta-color>
103	
104	- **Debug key**: in K8s RBAC, 403 means the ServiceAccount exists but lacks permission; 401 means the token is expired or invalid.
105	
106	#### <hepta-color type="text" color="yellow">404 Not Found</hepta-color>
107	
108	The classic error. In K8s it often means a misspelled Service name, or Ingress path routing did not match.
109	
110	---


## Note 91: Request and Response Model

- id: `95ab6bf9-976c-4937-a97f-854223f00d76`
- last_edited: 2026-10-11T01:44:22.245Z
- whiteboards: DevOp All, Computer Network Foundation

card "Request and Response Model" [95ab6bf9-976c-4937-a97f-854223f00d76] 101 lines
--- Databases ---
none
1	# Request and Response Model
2	
3	You walk into a restaurant. Before ordering, you make sure a waiter is there and ready to listen. Then you state the need in some language: "Beef noodle soup, no green onion." The waiter goes to the kitchen and returns with food and a receipt: "Done" or "Sorry, out of beef."
4	
5	On the network, TCP (transport) is the pipe that proves you and the waiter are connected. **HTTP (HyperText Transfer Protocol)** is the language and format you speak.
6	
7	If you do not know that structure, when Kubernetes `Ingress` forwarding fails or an API errors, you cannot tell a broken pipe (L4) from bad content (L7). This part unpacks every detail of an HTTP conversation.
8	
9	## HTTP vs TCP: language and pipe
10	
11	Earlier we built intuition for encapsulation and layers. Remember the nesting dolls?
12	
13	<hepta-color type="text" color="yellow">HTTP is OSI layer 7 (application), closest to the user. HTTP does not carry packets from Taiwan to the US — that is IP. It does not ensure no loss — that is TCP.</hepta-color>
14	
15	<hepta-color type="text" color="yellow">**HTTP's only job is to define data format.**</hepta-color>
16	
17	When you type a URL or run `kubectl get pods`, the OS first builds a TCP connection (three-way handshake). Once that reliable virtual pipe exists, HTTP can flow.
18	
19	TCP is a clear tube; HTTP is fixed-format letters sliding through it. Without a successful TCP handshake, HTTP never sends. That is why we debug with `ping` or `telnet` (L3/L4) first, then HTTP Request/Response (L7).
20	
21	## The full HTTP loop: one complete assignment
22	
23	<hepta-color type="text" color="yellow">HTTP's core is the </hepta-color><hepta-color type="text" color="yellow">**request-response model**</hepta-color>. 
24	
25	Classic Q&A: the client (browser or `curl`) starts; the server (Nginx or Kubernetes API Server) answers.
26	
27	<hepta-color type="text" color="yellow">A full HTTP loop usually has four stages:</hepta-color>
28	
29	1. <hepta-color type="text" color="yellow">**Open TCP**</hepta-color><hepta-color type="text" color="yellow">: Client three-way handshake. On modern HTTPS this also includes TLS handshake (later).</hepta-color>
30	
31	2. <hepta-color type="text" color="yellow">**Send Request**</hepta-color><hepta-color type="text" color="yellow">: Client sends a formatted letter down the pipe saying what it wants.</hepta-color>
32	
33	3. <hepta-color type="text" color="yellow">**Process and return Response**</hepta-color><hepta-color type="text" color="yellow">: Server reads the letter, understands, works (e.g. hits a database), and sends another letter back.</hepta-color>
34	
35	4. <hepta-color type="text" color="yellow">**What happens to the connection**</hepta-color><hepta-color type="text" color="yellow">：</hepta-color>
36	
37	   - <hepta-color type="text" color="yellow">**In early HTTP/1.0, TCP closed after each Request-Response pair — wasteful.**</hepta-color>
38	
39	   - <hepta-color type="text" color="yellow">**Today (HTTP/1.1+) we usually use Keep-Alive so the TCP pipe stays open for more requests, skipping repeated handshakes.**</hepta-color>
40	
41	Like calling support and, after one issue, saying "one more thing..." instead of hanging up and redialing.
42	
43	## Anatomy of an HTTP Request
44	
45	An HTTP request is plain text structured into three parts: **Request Line**, **Headers**, and **Body** (separated from headers by an empty line).
46	
47	```http
48	POST /api/v1/namespaces/default/pods HTTP/1.1
49	Host: k8s-api.example.com
50	User-Agent: kubectl/v1.28.0
51	Authorization: Bearer eyJhbGciOi...
52	Content-Type: application/json
53	Content-Length: 54
54	
55	{
56	  \"kind\": \"Pod\",
57	  \"metadata\": { \"name\": \"demo-pod\" }
58	}
59	```
60	
61	## Anatomy of an HTTP Response
62	
63	The server's response follows the exact same three-part structure: **Status Line**, **Headers**, and **Body**.
64	
65	```http
66	HTTP/1.1 201 Created
67	Date: Sun, 11 Oct 2026 01:43:00 GMT
68	Server: nginx/1.25.0
69	Content-Type: application/json
70	Content-Length: 68
71	
72	{
73	  \"status\": \"Success\",
74	  \"message\": \"Pod demo-pod created successfully\"
75	}
76	```
77	
78	## <hepta-color type="text" color="yellow">HTTP amnesia: stateless</hepta-color>
79	
80	<hepta-color type="text" color="yellow">HTTP has a famous property: </hepta-color><hepta-color type="text" color="yellow">**stateless**</hepta-color><hepta-color type="text" color="yellow">。</hepta-color>
81	
82	<hepta-color type="text" color="yellow">Meaning: </hepta-color><hepta-color type="text" color="yellow">**the server does not remember prior requests.**</hepta-color><hepta-color type="text" color="yellow"> Every request is new; there is no context memory.</hepta-color>
83	
84	<hepta-color type="text" color="yellow">Sounds dumb? You click Add to cart, then Checkout. If the server forgot who you are, whose cart does it bill?</hepta-color>
85	
86	### <hepta-color type="text" color="yellow">Why design it stateless?</hepta-color>
87	
88	<hepta-color type="text" color="yellow">The design is for </hepta-color><hepta-color type="text" color="yellow">**horizontal scaling**</hepta-color><hepta-color type="text" color="yellow">。
89	In DevOps many Pods run the same service. If HTTP were stateful, the client would have to stick to one Pod that rememb</hepta-color>ers login. Load balancing would be painful.
90	
91	<hepta-color type="text" color="yellow">Because HTTP is stateless, any Pod can handle any request alone. That plug-and-play property is how cloud-native systems scale.</hepta-color>
92	
93	### Then how do we remember?
94	
95	To fill the gap we built a few mechanisms:
96	
97	1. <hepta-color type="text" color="yellow">**Cookies and Session**</hepta-color><hepta-color type="text" color="yellow">: Server hands the client a ticket (Cookie) in the response; the client sends it next time so the server recognizes you.</hepta-color>
98	
99	2. <hepta-color type="text" color="yellow">**Token (e.g. JWT)**</hepta-color><hepta-color type="text" color="yellow">: More common in microservices and Kubernetes. Each request carries an encrypted credential in the </hepta-color><hepta-color type="text" color="yellow">`Authorization`</hepta-color><hepta-color type="text" color="yellow"> header. The server verifies it every time and stores no state in memory.</hepta-color>
100	
101	Like a clinic: the doctor does not remember you (stateless), but you hold a chart (Cookie/Token). One look and they continue.


## Note 92: Short note and QA

- id: `a47ae521-c606-4b3d-a22d-180284cb7960`
- last_edited: 2026-10-10T07:15:33.230Z
- whiteboards: DevOp All, Computer Network Foundation

card "Short note and QA" [a47ae521-c606-4b3d-a22d-180284cb7960] 173 lines
--- Databases ---
none
1	# Short note and QA
2	
3	### [Request and Response Model](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/95ab6bf9-976c-4937-a97f-854223f00d76)
4	
5	- HTTP is an application-layer protocol on TCP, following a request-and-response cycle.
6	
7	- A request has a request line, headers, and body. A response has a status line, headers, and body.
8	
9	- HTTP is stateless:
10	
11	  - The server does not remember request context. Each request is independent.
12	
13	  - That makes services easy to scale horizontally and is a foundation of cloud-native systems such as Kubernetes Pods.
14	
15	  - State is usually kept with cookies or tokens (JWT).
16	
17	### [Methods and Status Codes](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/f2971263-21d4-44af-8970-23826426a83a)
18	
19	- HTTP methods express client intent, with safety and idempotency:
20	
21	  - GET: safe and idempotent. Read only.
22	
23	  - PUT: idempotent. Fully replace the resource at a location (like assignment $x = 10$）。
24	
25	  - POST: not idempotent. Create a resource. Repeating it may create duplicates.
26	
27	- Status codes are the server's diagnosis, 1xx through 5xx:
28	
29	  - 401 Unauthorized vs 403 Forbidden: the first is authentication failure (who are you?); the second is authorization failure (you have no permission).
30	
31	  - 502 Bad Gateway: the gateway reached the backend but got an invalid response. Often a crashed Pod or a connection issue.
32	
33	  - 504 Gateway Timeout: the gateway reached the backend but timed out. Often a slow backend (slow query).
34	
35	### [HTTP Headers](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/602b0846-c1f4-465b-a655-3d8035b8e49b)
36	
37	- Headers are metadata that define the conversation environment.
38	
39	- Host header: the soul of modern web and Kubernetes Ingress.
40	
41	  - One IP can host many domains (virtual hosting).
42	
43	  - The Ingress Controller routes by this header to the right Service.
44	
45	- Content negotiation: `Content-Type` (sender format) and `Accept` (receiver expected format) keep parsing consistent.
46	
47	- Cache and performance:`Cache-Control` sets cache duration; `ETag` is a resource fingerprint. A 304 avoids unnecessary data transfer.
48	
49	### [HTTP/1.1 vs HTTP/2](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/50f9f650-a53b-4876-834b-a6524afdb9aa)
50	
51	- HTTP/1.1 bottlenecks:
52	
53	  - Plain text is slow to parse.
54	
55	  - L7 head-of-line blocking: requests and responses must be processed in order.
56	
57	- HTTP/2 core reforms:
58	
59	  - Binary framing: chop the payload into frames that machines parse faster.
60	
61	  - Multiplexing: many streams on one TCP connection, identified by Stream ID, which removes L7 blocking.
62	
63	  - Header compression (HPACK): dynamic and static tables cut repeated-header bandwidth.
64	
65	- Kubernetes use: gRPC, kubectl, and the API server all rely heavily on HTTP/2.
66	
67	### [Why HTTPS Is Needed](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/08da616b-c9bf-44f2-b03d-b962ef4a27ae)
68	
69	- HTTP is plaintext, like a postcard. Every hop can read and change it.
70	
71	- Three HTTP threats:
72	
73	  - **Eavesdropping**: breaks confidentiality. Packet sniffing steals private data.
74	
75	  - **Tampering**: breaks integrity. A MITM can inject malware or change content.
76	
77	  - **Impersonation**: breaks authentication. The user may connect to a fake server.
78	
79	- HTTPS answers (CIA triad):
80	
81	  - **Encryption**: turn plaintext into ciphertext for confidentiality.
82	
83	  - **Message integrity**: a digital fingerprint so data cannot be changed unnoticed.
84	
85	  - **Certificates**: prove the server's identity.
86	
87	### [Certificates and the CA Trust Chain](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/8e7eea12-cdf4-406b-8f0f-e45bc022fd3e)
88	
89	- **X.509 certificate structure**: subject (domain), public key (key exchange), issuer (CA), validity, and digital signature.
90	
91	- **Certificate authority (CA)**: a trusted third party that vets the applicant and issues the certificate. Types include DV, OV, and EV.
92	
93	- **Chain of trust**：
94	
95	  - **Root CA**: the end of trust. Root certificates are preloaded in the OS or browser root store.
96	
97	  - **Intermediate CA**: issues certificates as an agent so the root CA private key stays offline and revocation is easier.
98	
99	  - **Leaf Certificate**: the leaf certificate given to the website.
100	
101	- **Self-signed certificates**: not issued by a trusted CA. Browsers warn because they cannot chain to the root store.
102	
103	### [TLS Handshake Steps](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/b33c26aa-1cd3-4e02-a756-1bbb7b1b1e3f)
104	
105	- Two jobs of the TLS handshake:**authentication** and **key agreement**。
106	
107	- **TLS 1.2 flow (2-RTT)**：
108	
109	  - **Phase 1**: ClientHello and ServerHello exchange Client/Server Random and agree a cipher suite.
110	
111	  - **Phase 2**: the server presents its certificate; the client verifies identity.
112	
113	  - **Phase 3**: the client encrypts a Pre-master Secret with the server public key.
114	
115	  - **Phase 4**: both sides combine three randoms into a symmetric session key, switch to the encrypted channel, and finish.
116	
117	- **TLS 1.3 evolution (1-RTT)**: handshake in one round trip, plus 0-RTT resumption, lower latency and stronger security.
118	
119	### [Symmetric and Asymmetric Encryption](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/card/dca640e4-9831-49ca-9c46-0b78553793f2)
120	
121	- **Symmetric encryption**: same key for encrypt and decrypt (AES). Fast, but hard to exchange the key safely on a public network.
122	
123	- **Asymmetric encryption**: public key encrypts, private key decrypts (RSA, ECC). Solves key distribution and authentication, but is computationally heavy.
124	
125	- **TLS hybrid strategy**：
126	
127	  - Handshake uses **asymmetric encryption** to exchange secrets safely.
128	
129	  - Data transfer uses **symmetric encryption** for the actual traffic.
130	
131	- **DevOps practice**: reuse connections and prefer ECC certificates to keep TLS cost down.
132	
133	## Q&A
134	
135	- **Q:** Are HTTP method semantics (GET is read-only) mandatory?
136	
137	  - They are a convention, not a law. You can implement delete with GET, but it is catastrophic.
138	
139	  - Browser prefetch and crawler indexing assume GET is safe. Breaking that can delete data or make alerts meaningless.
140	
141	- **Q:** What does Bearer in Bearer Token mean? Why the space?
142	
143	  - Bearer means the holder. Anyone holding the token has the permission.
144	
145	  - The header format is `Authorization: <scheme> <credentials>`. The space separates scheme from credential so the server parser can split them.
146	
147	- **Q:** Is an HTTP/2 REST API still human-readable?
148	
149	  - For developers, tools such as Chrome DevTools reconstruct frames as text, so it still feels human-readable.
150	
151	  - On the wire it is binary 0s and 1s, not HTTP/1.1 plaintext strings.
152	
153	- **Q:** Why does curling an Ingress IP return 404?
154	
155	  - Ingress routes on the Host header. Curling the IP sets Host to that IP.
156	
157	  - Fix it with `curl -H "Host: <domain>" <IP>` to simulate the right routing label.
158	
159	- **Q:** Is there only one company that issues certificates (CA)?
160	
161	  - Certificates are a global market, not a monopoly.
162	
163	  - It includes commercial giants such as DigiCert, Let's Encrypt (nonprofit, automated), and cloud providers such as AWS ACM.
164	
165	  - The bar to be a CA is inclusion in Apple, Google, and other root stores.
166	
167	- **Q:** Does every HTTP request need a new TLS handshake?
168	
169	  - No. Frequent handshakes would be very expensive.
170	
171	  - Mainly **HTTP Keep-Alive** keeps the TCP connection open and reuses the encrypted channel.
172	
173	  - **TLS session resumption** and **TLS 1.3 0-RTT** also let a reconnect skip a full handshake.


## Note 93: Questions worth keeping

- id: `5dbb82c5-fbde-4405-bdbe-773ccb51d265`
- last_edited: 2026-10-10T07:00:12.219Z
- whiteboards: DevOp All, Computer Network Foundation

card "Questions worth keeping" [5dbb82c5-fbde-4405-bdbe-773ccb51d265] 13 lines
--- Databases ---
none
1	# Questions worth keeping
2	
3	- **Q: What does the hierarchy actually solve?** It isolates names and delegates ownership. The same leaf (www) does not collide under different parents, and no one waits on a global editor.
4	
5	- **Q: Why is 8.8.8.8 not a bottleneck?** Cache plus Anycast. Answers sit for the TTL, and the query hits the nearest node, not one machine.
6	
7	- **Q: Why are overseas users still on the old IP 18 hours later, when TTL was 24 hours?** Their resolver cached the old answer and will not ask again until that TTL dies. Wait the remaining 6 hours. Lowering TTL on cutover day does not shorten caches that already stored 24 hours.
8	
9	- **Q: Why does a fat answer switch from UDP to TCP?** Over 512 bytes with no EDNS0 size, the server sets TC. The client drops the truncated UDP reply and repeats the query on TCP/53.
10	
11	- **Q: Why does ndots:5 make api.github.com slow inside a Pod?** The name has fewer than 5 dots, so the stub first tries api.github.com.<namespace>.svc.cluster.local and the other search suffixes. Each miss is an NXDOMAIN to CoreDNS before the real query.
12	
13	- **Q: Does cluster DNS use the same tree?** Yes. cluster.local is the root, svc is the type, namespace separates environments, and the Service name is the leaf.


# Capture note

Appended 93 note reads. Journals in the top 100 were listed in the topology section and not read.

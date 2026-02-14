# Kiến thức

### 1. Khái niệm Orchestration (Điều phối)

Khác với **Tự động hóa (Automation)** chỉ tập trung vào một khía cạnh (như cài đặt server), **Orchestration** có phạm vi rộng hơn nhiều.

- **Định nghĩa:** Orchestration là hệ thống tự động hóa việc phối hợp nhiều hệ thống con (mạng, lưu trữ, và container) để cấu hình, triển khai, vận hành và giám sát một dịch vụ phần mềm,.
- **Mục tiêu chính:**
    1. **Dynamic scaling:** Tự động tăng/giảm số lượng bản sao (replicas) theo nhu cầu.
    2. **Coordination:** Phân phối tải và đặt container lên các server vật lý phù hợp để tối ưu hiệu năng.
    3. **Resilience:** Tự động phục hồi khi có lỗi (ví dụ: khởi động lại container bị crash).

### 2. Kubernetes (K8s) - Công nghệ cốt lõi

Chương này sử dụng **Kubernetes** làm ví dụ điển hình cho hệ thống Container Orchestration.

- **Mô hình Cluster:** Kubernetes vận hành dựa trên mô hình Cluster, bao gồm một **Master Node** (chạy phần mềm điều khiển) và các **Worker Nodes** (nơi chạy các ứng dụng thực tế),.
- **Pods:** Kubernetes không quản lý trực tiếp container đơn lẻ mà quản lý **Pod**. Pod là đơn vị nhỏ nhất, bao gồm một hoặc một nhóm các container gắn kết chặt chẽ với nhau (thường chung mạng, chung lưu trữ),.

### 3. Kiến trúc phần mềm của Kubernetes (Rất quan trọng)

Hệ thống được chia thành hai nhóm thành phần chính:

#### A. Control Plane (Chạy trên Master Node),

Đây là "bộ não" của hệ thống, bao gồm 5 thành phần:

1. **API Server:** Cổng giao tiếp trung tâm. Mọi lệnh từ người dùng (qua `kubectl`) hoặc giao tiếp giữa các thành phần nội bộ đều đi qua đây.
2. **Scheduler:** Chịu trách nhiệm chọn Node phù hợp nhất để chạy một Pod mới.
3. **Cluster State Store (etcd):** Cơ sở dữ liệu lưu trữ toàn bộ trạng thái của cluster (cấu hình, node nào đang chạy pod nào...). Nó sử dụng key-value store,.
4. **Controller Manager:** Chạy các vòng lặp điều khiển (control loops) để đảm bảo trạng thái thực tế khớp với trạng thái mong muốn (ví dụ: Replication Controller).
5. **Cloud Controller Manager:** Giao tiếp với API của nhà cung cấp đám mây (AWS, Azure...) để quản lý tài nguyên vật lý như Load Balancer hay ổ cứng.

#### B. Worker Node Components (Chạy trên các Node làm việc),

1. **Kubelet:** "Thuyền trưởng" của mỗi Node. Nó giao tiếp với API Server để nhận lệnh và quản lý các Pod trên Node đó.
2. **Service Proxy (kube-proxy):** Quản lý quy tắc mạng, giúp chuyển tiếp gói tin đến đúng Pod.
3. **Container Runtime:** Phần mềm thực thi container (ví dụ: Docker Engine).

### 4. Các tính năng chính (Features)

Kubernetes cung cấp 7 tính năng chính giúp đơn giản hóa việc vận hành,:

1. **Service naming and discovery:** Đặt tên và tìm kiếm dịch vụ (DNS).
2. **Load balancing:** Cân bằng tải giữa các bản sao của container.
3. **Storage orchestration:** Tự động gắn (mount) các hệ thống lưu trữ.
4. **Optimized placement:** Tối ưu hóa việc đặt container vào server nào.
5. **Automated recovery:** Tự động thay thế hoặc khởi động lại container bị lỗi.
6. **Secret/Configuration management:** Quản lý mật khẩu và cấu hình an toàn.
7. **Automated rollouts/rollbacks:** Cập nhật phiên bản mới hoặc quay lại phiên bản cũ một cách tự động và an toàn.

## Tại sao lại cần điều phối "Orchestration"
>[!Question] Vấn đề
>Tại sao tự động hóa (Automation) là chưa đủ?

Ở chapter 9, vấn đề tự động hóa đã được đề cập. Tuy nhiên ở mức độ tự động hóa thì chỉ có thể thực hiện tự động hóa được các tác vụ rời rạc. chính vì vấn đề rời rạc cũng như vòng đời của tự động hóa nên chính vì vậy mà điều phối (Orchestration) được ra đời. **Điểm khác biệt** giữa Orchestration và Automation là nằm ở tính chất phạm vi và vòng đời của đối tượng quản lý.

>[!Question] Vấn đề của Container là gì?

Container có tính chất "phù du" (ephemeral) với tuổi thọ ngắn. Khác với VM có tính chất bán vĩnh viễn (khi nào xóa nó mới mất) thì Container được sinh ra để xử lý một yêu cầu rồi biến mất

Khi lên đến hàng nghìn Container thì con người không thể tiến hành cấu hình mạng (IP, Port) hay gắn ổ cứng (Storage Mount) kịp thời cho từng Container.

>[!Solution] Giải pháp
>Orchestration không chỉ chạy phần mềm mà còn là điều phối (coordinate) tất cả các hệ thống con từ:
> + Triển khai (deployment)
> + Cấu hình mạng (networking)
> + Quản lý lưu trữ (storage)

Trong đó nổi bật hơn cả chính là hệ thống điêu phối Kubernetes (K8S) với mô hình Cluster (Cụm) và đơn vị Pod

## Kubernetes
Kubernetes được biết đến là phần mềm điều phối (Orchestration) phổ biến nhất hiện nay
Mô hình Kubernetes bao gồm các thành phần chính sau
### Mô hình Cluster (Cụm)
* Cluster trong K8s chia tách thành hai role rất rõ ràng:
*1 Chủ sở hữu (Owner/ Operator)*: Sử dụng Control Plane để vận hành cụm
_2 Người dùng (User/Dev)_: Sử dụng các phần mềm để chạy ứng dụng trên các worker node. Điểm mấu chốt là người dùng không biết được ứng dụng đang được chạy trên máy chủ vật lý thật nào, họ chỉ tương tác với API của cụm
### Pod - Đơn vị nguyên tử (Atomic Unit)
Tại sao K8s quản lý Pod chứ không phải Container đơn lẻ
**Tightly Coupled**: Pod đại diện cho một ứng dụng hoàn chỉnh, mà một ứng dụng có thể cần nhiều container để có thể hoạt động hỗ trợ nhau (Một container webserver, 1 container log shipper)
**Chia sẻ mạng**: Tất cả các container trong một pod chia sẻ chung **một địa chỉ IP** và một không gian cổng (localhost). Điều này tương tự với các tiến trình chạy trên một máy tính.
**Late Binding**: K8s sử dụng template (manifest) để tạo Pod. Việc thay đổi templates chỉ ảnh hưởng đến các Pod mới, không ảnh hưởng đến các Pod đang chạy, giúp quản lý phiên bản an toàn
**Init Containers**: Đây là tính nawg quan trọng để đảm bảo tiền điều kiện **preconditions**. Init Container chạy và phải hoàn tất thành công trước khi các container chính khởi động. Điều này đảm bảo Pid hoặc chạy hoàn hảo hoặc không chạy gì cả, tránh tình trạng lỗi lửng lơ, lãng phí


Chào bạn, dựa trên các tài liệu đã cung cấp, đặc biệt là **Chương 10: Orchestration** trong cuốn _The Cloud Computing Book_ và các slide bài giảng, tôi sẽ tiến hành **phân tích sâu** nội dung chương này.

Chúng ta sẽ không chỉ dừng lại ở các định nghĩa, mà sẽ đi sâu vào kiến trúc nội tại, cơ chế giao tiếp và cách Kubernetes giải quyết các bài toán phức tạp của hệ thống phân tán.

---

### 3. Phân Tích Kiến Trúc Control Plane (Bộ não)

Control Plane chạy trên Master Node và gồm 5 thành phần phối hợp chặt chẽ,. Điểm đặc biệt là cơ chế giao tiếp "hình sao" qua API Server.

1. **API Server (Cổng giao tiếp trung tâm):**
    - Đây là "tổng đài". Mọi giao tiếp nội bộ (giữa Scheduler và Kubelet) hoặc bên ngoài (từ Admin qua `kubectl`) đều **bắt buộc** phải đi qua API Server. Không có thành phần nào nói chuyện trực tiếp với nhau,.
2. **Cluster State Store (etcd):**
    - Nơi duy nhất lưu trữ trạng thái của cụm ("Source of Truth"). K8s sử dụng **etcd** (key-value store) vì tính nhất quán cao (consensus algorithm), đảm bảo dữ liệu không bị mất ngay cả khi có sự cố.
3. **Scheduler (Người sắp xếp):**
    - Nhiệm vụ: Chọn Node cho Pod.
    - Cơ chế: Nó theo dõi (watch) các Pod mới chưa được gán Node, tính toán tài nguyên và ràng buộc, sau đó "bind" Pod vào Node thích hợp nhất,.
4. **Controller Manager (Người điều hành):**
    - Đây là nơi chạy các "Vòng lặp điều khiển" (Control Loops) để đảm bảo trạng thái thực tế khớp với trạng thái mong muốn (Desired State). Ví dụ: Nếu bạn muốn 3 replicas, Controller Manager sẽ liên tục kiểm tra, nếu thấy chỉ có 2, nó sẽ tạo thêm 1.
5. **Cloud Controller Manager:**
    - Cầu nối với nhà cung cấp mây (AWS, Azure...). Nó giúp K8s hiểu được topology của hạ tầng vật lý bên dưới (ví dụ: tạo Load Balancer thật của AWS khi K8s yêu cầu).

---

### 4. Phân Tích Kiến Trúc Worker Node (Cơ bắp)

Worker Node là nơi thực sự chạy tải (workload). Nó có 3 thành phần chính,:

1. **Kubelet (Thuyền trưởng của Node):**
    - Là "agent" quan trọng nhất. Nó nhận lệnh từ API Server (trên Master), tương tác với Container Runtime để khởi tạo/dừng container.
    - Nó giám sát sức khỏe của Pod và báo cáo ngược lại cho Master. Nó sử dụng **cAdvisor** để thu thập thống kê tài nguyên (CPU, RAM).
2. **Service Proxy (kube-proxy):**
    - Chịu trách nhiệm về **Mạng**. Nó cấu hình các quy tắc mạng trên Node (thường dùng `iptables` của Linux) để đảm bảo gói tin gửi đến địa chỉ ảo của Service sẽ được chuyển tiếp đúng đến các Pod backend.
3. **Container Runtime:**
    - K8s không tự chạy container, nó nhờ một runtime (như Docker Engine, containerd) làm việc đó.

---

### 5. Các Tính Năng Cao Cấp & Cơ Chế Hoạt Động

Chương 10 nhấn mạnh các tính năng giúp K8s trở thành chuẩn mực công nghiệp:

- **Service & Load Balancing:**
    - K8s giải quyết vấn đề "địa chỉ IP thay đổi liên tục" của Pod bằng khái niệm **Service**. Service cung cấp một địa chỉ IP ảo ổn định. `kube-proxy` sẽ cân bằng tải traffic từ IP ảo này tới các Pod thực tế phía sau,.
- **Self-healing (Tự phục hồi):**
    - Nếu container chết, Kubelet khởi động lại nó. Nếu cả Node chết, Controller Manager (trên Master) sẽ phát hiện và tạo Pod thay thế trên Node khác.
- **Automated Rollouts/Rollbacks:**
    - K8s cho phép cập nhật ứng dụng dần dần (rolling update). Nó thay thế từng Pod cũ bằng Pod mới, đảm bảo zero-downtime. Nếu phiên bản mới lỗi, nó có thể tự động quay lại (rollback) phiên bản cũ.
- **Quản lý cấu hình & Bí mật (Secrets):**
    - Tách biệt mã nguồn (Container Image) khỏi cấu hình. Cùng một Image có thể chạy ở Dev, Test, Prod chỉ bằng cách thay đổi ConfigMap hoặc Secret được mount vào.

### Tổng kết Phân tích

Chương 10 cho thấy Orchestration không chỉ là việc "bật tắt" container. Đó là một hệ thống phức tạp dựa trên **Mô hình Khai báo (Declarative Model)**:

- Con người khai báo: "Tôi muốn 3 bản sao của App A".
- Hệ thống (K8s Control Plane): Liên tục làm việc để biến điều đó thành sự thật, bất chấp lỗi phần cứng hay mạng xảy ra.

Sự tách biệt rõ ràng giữa **Control Plane** (quản lý trạng thái) và **Worker Node** (thực thi), cùng với cơ chế giao tiếp qua trung gian **API Server**, chính là thiết kế giúp K8s có khả năng mở rộng và độ tin cậy cực cao.


### 1. Automation (Tự động hóa)

Tự động hóa tập trung vào việc thực hiện một nhiệm vụ cụ thể hoặc một tập hợp nhỏ các nhiệm vụ mà không cần sự can thiệp của con người.

- **Phạm vi:** Hẹp. Nó thường tập trung vào một khía cạnh cụ thể của hoạt động trung tâm dữ liệu.
- **Mục tiêu:** Giúp các tác vụ trở nên dễ dàng và nhanh chóng hơn cho con người thực hiện, giảm thiểu lỗi do con người (như lỗi đánh máy khi cấu hình),.
- **Nguồn gốc:** Thường bắt nguồn từ việc mô phỏng lại các quy trình thủ công mà con người từng làm (ví dụ: các bước triển khai một VM),.
- **Đặc điểm:** Các công cụ tự động hóa thường hoạt động độc lập để giải quyết từng nhiệm vụ nhỏ (ví dụ: một công cụ để cấu hình mạng, một công cụ khác để cài đặt OS).

### 2. Orchestration (Điều phối)

Điều phối là tự động hóa với phạm vi lớn hơn nhiều. Nó không chỉ thực hiện một tác vụ mà là phối hợp nhiều hệ thống con để cấu hình, triển khai, vận hành và giám sát trọn vẹn một dịch vụ phần mềm.

- **Phạm vi:** Rộng và bao quát. Nó điều phối tất cả các hệ thống con cần thiết (mạng, lưu trữ, tính toán) để vận hành một dịch vụ.
- **Mục tiêu:** Tự động hóa toàn bộ quy trình cung cấp dịch vụ (Full automation of a service), bao gồm triển khai, cấu hình, cập nhật, phục hồi lỗi và tự động mở rộng (autoscaling),.
- **Cơ chế:** Orchestration lắp ghép các "khối xây dựng" (building blocks) - như container hoặc VM - để tạo thành các dịch vụ phức tạp.
- **Tính năng nâng cao:** Không giống như tự động hóa đơn thuần, hệ thống điều phối (như Kubernetes) xử lý các vấn đề phức tạp như:
    - **Mở rộng động (Dynamic scaling):** Tự động tăng/giảm số lượng bản sao dựa trên nhu cầu.
    - **Phối hợp trên nhiều máy chủ:** Cân bằng tải và đặt các container lên các máy chủ vật lý phù hợp.
    - **Khả năng phục hồi (Resilience):** Tự động khởi động lại hoặc thay thế các thành phần bị lỗi để đảm bảo tính sẵn sàng.

### Tóm lại

Bạn có thể hình dung sự khác biệt như sau:

- **Automation** giống như việc chế tạo một cánh tay robot để tự động gạt cần số xe hơi (bắt chước hành động đơn lẻ của con người).
- **Orchestration** giống như một hệ thống xe tự lái hoàn chỉnh, nơi hệ thống không chỉ gạt cần số mà còn phối hợp phanh, ga, vô lăng và cảm biến để đưa xe đến đích an toàn mà không cần can thiệp.

> **Trích dẫn đắt giá từ tài liệu:** "Không giống như một công cụ tự động hóa tập trung vào một khía cạnh của hoạt động trung tâm dữ liệu, một hệ thống điều phối phối hợp tất cả các hệ thống con cần thiết để vận hành một dịch vụ, bao gồm triển khai container và cấu hình cả giao tiếp mạng và lưu trữ".

# So sánh
Dựa trên các tài liệu được cung cấp (đặc biệt là _The Cloud Computing Book - Chapter 10 & 13_, và các slide _Orchestration_), tôi sẽ tiếp tục phân tích sâu hơn về cơ chế vận hành của Kubernetes (K8s) và thực hiện so sánh với các công cụ/hệ thống khác.

### 1. Phân tích sâu: Cơ chế "Controller Pattern" trong Kubernetes

Để hiểu tại sao K8s có thể quản lý hàng nghìn container một cách tự động, ta phải hiểu **Mô hình Bộ điều khiển (Controller Paradigm)** và **Vòng lặp điều khiển (Control Loop)**,,.

#### A. Nguyên lý hoạt động (Intent-based / Declarative)

K8s không hoạt động theo kiểu "mệnh lệnh" (Imperative - làm bước A rồi làm bước B). Nó hoạt động theo kiểu "khai báo" (Declarative - Tôi muốn kết quả là X).

- **Người dùng:** Khai báo **Trạng thái mong muốn (Desired State)** trong file YAML (ví dụ: "Tôi muốn 3 replicas của Nginx").
- **Hệ thống:** Liên tục so sánh **Trạng thái thực tế (Actual State)** với Trạng thái mong muốn. Nếu có sự chênh lệch (diff), nó sẽ thực hiện hành động điều chỉnh,.

#### B. Vấn đề Trễ (Hysteresis) và Sự ổn định

Một thách thức lớn của hệ thống tự động là **Hysteresis** (Độ trễ vật lý).

- Ví dụ: Khi tải CPU tăng, K8s ra lệnh tạo thêm Pod mới. Nhưng Pod cần thời gian để tải image và khởi động. Trong thời gian đó, CPU vẫn cao, nếu bộ điều khiển không tính đến độ trễ này, nó có thể tiếp tục ra lệnh tạo thêm Pod thừa thãi, gây ra sự bất ổn định (instability),,.
- **Giải pháp:** K8s sử dụng các thuật toán "stabilization window" (cửa sổ ổn định) để tránh việc scale-out/scale-in liên tục quá nhanh.

#### C. Kiến trúc hướng sự kiện (Event-Driven Implementation)

Thay vì liên tục hỏi (polling) API Server "Trạng thái hiện tại là gì?" (gây tốn tài nguyên), K8s sử dụng cơ chế hướng sự kiện,:

1. **Informer/Watcher:** Lắng nghe các thay đổi từ API Server.
2. **Workqueue:** Khi có thay đổi (ví dụ: một Pod bị chết), một sự kiện được đẩy vào hàng đợi.
3. **Worker:** Lấy sự kiện từ hàng đợi và thực hiện hàm `Reconcile` (Hòa giải) để đưa hệ thống về trạng thái mong muốn,.

#### D. Khả năng mở rộng: CRD và Custom Controllers

K8s không chỉ quản lý các tài nguyên có sẵn (Pod, Service). Nó cho phép người dùng tự định nghĩa tài nguyên mới thông qua **Custom Resource Definition (CRD)**,.

- Ví dụ: Bạn có thể định nghĩa một tài nguyên tên là `Database`. Khi bạn tạo một object `Database`, một **Custom Controller** do bạn viết sẽ tự động tạo các Pod, Persistent Volume, và ConfigMap cần thiết để chạy database đó. Đây là cơ sở của mô hình **Operator**.

---

### 2. So sánh Kubernetes với các công cụ điều phối khác

Dựa trên Source,,, việc so sánh K8s cần đặt trong ngữ cảnh của "khối xây dựng" (building blocks) mà nó quản lý.

#### A. Kubernetes vs. Docker Swarm (Container Orchestration)

Mặc dù tài liệu không đi sâu vào Swarm, nhưng có sự phân biệt rõ ràng:

- **Docker:** Là một "toolset" (bộ công cụ) và "runtime" để chạy container. Docker tập trung vào việc đóng gói và chạy ứng dụng đơn lẻ,.
- **Kubernetes:** Là hệ thống điều phối (orchestrator). K8s sử dụng Docker (hoặc containerd) làm runtime bên dưới để chạy container,.
- **So sánh:** Docker Swarm (công cụ điều phối tích hợp sẵn của Docker) đơn giản hơn, dễ dùng hơn nhưng thiếu các tính năng mạnh mẽ về quản lý trạng thái, tự phục hồi phức tạp và mở rộng như K8s. K8s đã trở thành tiêu chuẩn công nghiệp (de facto standard).

#### B. Kubernetes vs. OpenShift

OpenShift (của Red Hat) thực chất được xây dựng **dựa trên Kubernetes** (upstream), nhưng nó là một nền tảng đầy đủ (Platform) so với K8s là một công cụ (Engine),.

| Tiêu chí              | Kubernetes (K8s)                                                                       | OpenShift                                                                                           |
| :-------------------- | :------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------- |
| **Bản chất**          | Dự án nguồn mở, là "kernel" của hệ thống phân tán.                                     | Sản phẩm thương mại (Enterprise Ready), bao gồm K8s + các công cụ hỗ trợ.                           |
| **Cài đặt & Quản lý** | Phức tạp, người dùng phải tự cấu hình nhiều thành phần (Network, Ingress, Monitoring). | Tích hợp sẵn Monitoring (Prometheus), Logging, Networking, và có giao diện GUI mạnh mẽ ngay từ đầu. |
| **Bảo mật**           | Cấu hình bảo mật mặc định khá mở, người dùng phải tự thắt chặt.                        | Bảo mật chặt chẽ mặc định (ví dụ: không cho chạy container với quyền root).                         |
| **Build Image**       | K8s không build image, nó chỉ chạy image đã có.                                        | Có tích hợp sẵn Source-to-Image (S2I) và Jenkins cho quy trình CI/CD.                               |

#### C. Kubernetes vs. OpenStack

Sự khác biệt nằm ở cấp độ trừu tượng hóa tài nguyên:

- **OpenStack:** Là bộ điều phối cho **Máy ảo (VM)**. Đơn vị quản lý là VM. Nó tạo ra hạ tầng (IaaS) giống như AWS EC2.
- **Kubernetes:** Là bộ điều phối cho **Container**. Đơn vị quản lý là Pod. Nó chạy _trên_ hạ tầng (có thể là chạy trên OpenStack hoặc Bare-metal).
- **Mối quan hệ:** Hai hệ thống này thường bổ trợ nhau. OpenStack cung cấp các máy ảo, và K8s chạy bên trong các máy ảo đó để quản lý ứng dụng.

#### D. Kubernetes vs. Hadoop YARN

Sự khác biệt nằm ở mục đích sử dụng (Workload type),:

- **Hadoop YARN:** Tối ưu cho các tác vụ **xử lý dữ liệu lớn (Batch processing)** như MapReduce. Các tác vụ này có thời gian sống ngắn, chạy xong là tắt. YARN tập trung vào việc cấp phát tài nguyên tính toán cho các job này.
- **Kubernetes:** Tối ưu cho các **dịch vụ dài hạn (Long-running services)** như Web Server, Microservices. Mặc dù K8s có hỗ trợ `Job` và `CronJob`, nhưng thế mạnh của nó là duy trì trạng thái sẵn sàng (High Availability) cho các ứng dụng chạy liên tục.

### Tổng kết

- **Kubernetes** hiện là "hệ điều hành của đám mây", chịu trách nhiệm ghép nối các container lại thành một hệ thống nhất quán.
- Nó phức tạp hơn Docker đơn thuần vì nó giải quyết các bài toán ở quy mô lớn (scale, healing, scheduling).
- Nó khác biệt với OpenStack (quản lý VM) và YARN (quản lý Job BigData), nhưng có thể tích hợp hoặc chạy song song với các hệ thống này trong một Data Center hiện đại.


# ☸️ Chapter 10: Kubernetes Advanced Quiz Pool

**Tags:** #CloudComputing #Kubernetes #K8s #Orchestration #Chapter10 #AdvancedQuiz
**Số lượng:** 50 câu
**Mức độ:** Khó & Cực khó (Deep Dive)
**Tài liệu tham chiếu:** The Cloud Computing Book (Douglas Comer) & Kubernetes Internals

---

## Phần 1: Kiến trúc & Control Plane (Deep Dive)

### Câu 1
Trong kiến trúc Kubernetes, nếu thành phần **kube-scheduler** bị lỗi và ngừng hoạt động, điều gì sẽ xảy ra với một Pod mới được tạo (Pending Pod)?
A. Pod sẽ tự động được gán vào Node có tài nguyên trống nhiều nhất.
B. Pod sẽ bị xóa ngay lập tức và báo lỗi `SchedulerUnreachable`.
C. Pod sẽ kẹt ở trạng thái `Pending` mãi mãi cho đến khi scheduler hoạt động trở lại hoặc được gán thủ công.
D. `kubelet` sẽ tự động nhận Pod này về chạy để đảm bảo High Availability.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Scheduler là thành phần duy nhất có quyền quyết định gán Pod vào Node nào (bind pod to node). Nếu nó chết, Pod vẫn tồn tại trong API Server nhưng trường `nodeName` bị trống, do đó nó sẽ nằm ở trạng thái `Pending` vô thời hạn. Kubelet không tự ý lấy Pod nếu chưa được gán.

### Câu 2
Tại sao **etcd** lại là thành phần quan trọng nhất cần phải backup trong Cluster?
A. Vì nó lưu trữ logs của tất cả các container.
B. Vì nó là thành phần duy nhất lưu trữ trạng thái (Stateful) của toàn bộ Cluster.
C. Vì nó chứa Docker Images.
D. Vì nó quản lý việc định tuyến mạng giữa các Service.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Mọi thành phần khác (Scheduler, Controller Manager, API Server) đều là Stateless. Chỉ có **etcd** lưu trữ "Sự thật" (Single Source of Truth) về trạng thái cluster. Mất etcd là mất toàn bộ cluster.

### Câu 3
Cơ chế giao tiếp trong Control Plane tuân theo mô hình nào?
A. Các thành phần giao tiếp trực tiếp với nhau (Mesh).
B. Scheduler giao tiếp trực tiếp với Kubelet để ra lệnh chạy Pod.
C. Mọi thành phần chỉ giao tiếp với API Server (Hub-and-Spoke).
D. Kubelet giao tiếp trực tiếp với etcd để lấy cấu hình.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** API Server là trung tâm. Scheduler, Controller Manager, hay Kubelet **không bao giờ** nói chuyện trực tiếp với nhau hay với etcd. Chúng chỉ gọi API Server, và API Server là thành phần duy nhất được phép nói chuyện với etcd.

### Câu 4
Nếu **Master Node** (Control Plane) bị sập hoàn toàn (Offline), điều gì xảy ra với các Pod đang chạy trên Worker Nodes?
A. Tất cả Pod sẽ bị dừng ngay lập tức vì mất kết nối quản lý.
B. Các Pod vẫn tiếp tục chạy bình thường, nhưng không thể cập nhật, xóa hoặc tạo mới Pod.
C. Các Pod sẽ tự động reboot.
D. Worker Node sẽ tự bầu ra một Master mới từ các Pod đang chạy.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đây là tính tách biệt giữa Control Plane và Data Plane. Ứng dụng chạy ở Worker Node. Kubelet tại Worker Node vẫn giữ container chạy theo cấu hình cuối cùng nó nhận được. Tuy nhiên, tính năng "Self-healing" (tự tạo lại Pod nếu Pod chết) sẽ mất vì Controller Manager (trên Master) đã sập.

### Câu 5
Thành phần nào trên Worker Node chịu trách nhiệm giám sát và đảm bảo các container được mô tả trong PodSpec đang chạy?
A. kube-proxy.
B. Container Runtime (Docker/containerd).
C. kubelet.
D. cAdvisor.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** **kubelet** là "thuyền trưởng" trên mỗi node. Nó nhận PodSpec từ API Server và tương tác với Container Runtime để đảm bảo container chạy đúng như mô tả.

### Câu 6
Sự khác biệt cốt lõi về triết lý thiết kế giữa **Kubernetes** và các hệ thống quản lý cũ (như script) là gì?
A. Kubernetes nhanh hơn.
B. Kubernetes sử dụng mô hình Declarative (Khai báo) thay vì Imperative (Mệnh lệnh).
C. Kubernetes chỉ chạy được Linux Container.
D. Kubernetes không cần Database.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Trong K8s, bạn khai báo "Tôi muốn có 3 Pods" (Desired State). Hệ thống tự tìm cách đạt được điều đó (Reconciliation loop). Các hệ thống cũ thường dùng lệnh "Hãy chạy thêm 1 Pod nữa" (Imperative).

### Câu 7
Quy trình nào sau đây mô tả đúng luồng tạo một Pod?
A. User -> Scheduler -> API Server -> Kubelet.
B. User -> API Server -> Scheduler -> API Server -> Kubelet.
C. User -> Kubelet -> Docker.
D. User -> Controller Manager -> Scheduler -> Docker.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:**
> 1. User gửi yêu cầu tạo Pod tới API Server (lưu vào etcd).
> 2. Scheduler thấy Pod chưa có Node -> chọn Node -> Gửi lại API Server.
> 3. Kubelet (trên Node được chọn) thấy có việc -> gọi Container Runtime chạy Pod.

### Câu 8
**Cloud Controller Manager (CCM)** có vai trò gì?
A. Quản lý việc billing của Cloud.
B. Là cầu nối để K8s tương tác với API của Cloud Provider (như tạo AWS ELB, Azure Disk).
C. Quản lý user nội bộ của K8s.
D. Thay thế cho Kube-proxy.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** CCM cho phép K8s điều khiển tài nguyên của nhà cung cấp Cloud (ví dụ: Khi tạo Service type LoadBalancer, CCM sẽ gọi API của AWS/GCP để tạo Load Balancer vật lý).

### Câu 9
Trong K8s, **Namespace** KHÔNG cung cấp sự cô lập (Isolation) về khía cạnh nào theo mặc định?
A. Tên của Resource (Resource Names).
B. Quản lý quyền truy cập (RBAC).
C. Tài nguyên mạng (Network Traffic) giữa các Pod.
D. Quota tài nguyên (Resource Quotas).

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Mặc định, Pod ở Namespace A hoàn toàn có thể ping và gọi điện tới Pod ở Namespace B (Flat Network). Muốn chặn, phải dùng **Network Policies**. Namespace chỉ chia tách về mặt logic quản lý (Logical Partition).

### Câu 10
Để đảm bảo High Availability cho Control Plane, số lượng Master Node tối thiểu nên là bao nhiêu và tại sao?
A. 2 Nodes (Để có 1 dự phòng).
B. 3 Nodes (Để đảm bảo Quorum cho etcd khi bầu cử).
C. 4 Nodes (Cho chắc chắn).
D. 1 Node là đủ nếu Node đó mạnh.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** **etcd** sử dụng thuật toán đồng thuận Raft. Để chịu được lỗi của 1 node mà vẫn ghi được dữ liệu, cần tối thiểu 3 nodes (Công thức: `N/2 + 1`). Nếu dùng 2 nodes, khi 1 node chết, node còn lại chỉ chiếm 50% -> Không đạt đa số -> Cluster chuyển sang Read-only.

---

## Phần 2: Pod & Workload Management

### Câu 11
Tại sao Kubernetes chọn **Pod** làm đơn vị deploy nhỏ nhất chứ không phải **Container**?
A. Vì Docker Container quá chậm.
B. Để hỗ trợ mô hình nhóm các container liên quan chặt chẽ cần chia sẻ tài nguyên (Shared Context) như Network và Storage.
C. Vì thuật ngữ Pod nghe hay hơn.
D. Vì Kubernetes không hỗ trợ Docker.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Pod đại diện cho một "logical host". Nó cho phép mô hình **Sidecar Container** (ví dụ: 1 container log agent chạy kèm 1 container web app), chúng cần chia sẻ `localhost` và Volume.

### Câu 12
Hai container nằm trong cùng một Pod chia sẻ những gì?
A. File System (trừ khi dùng Volume) và PID.
B. Network Namespace (IP Address, Port space) và IPC Namespace.
C. CPU và RAM (không giới hạn).
D. Registry Credentials.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Mặc định quan trọng nhất là **Network**. Container A trong Pod có thể gọi Container B qua `localhost`. File system thì riêng biệt (trừ khi mount chung Volume).

### Câu 13
Trạng thái **CrashLoopBackOff** của Pod nghĩa là gì?
A. Pod đang chạy rất tốt.
B. Container trong Pod khởi động, nhưng bị crash ngay lập tức, K8s đang cố gắng khởi động lại nó với thời gian chờ tăng dần.
C. Node chứa Pod bị mất kết nối mạng.
D. Image của Pod không tồn tại.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Kubelet cố gắng restart container, nhưng nó cứ chết đi sống lại liên tục (do lỗi code, thiếu config...). K8s sẽ tăng thời gian chờ (BackOff) để tránh tốn tài nguyên CPU.

### Câu 14
**ReplicaSet** khác gì với **ReplicationController** (cũ)?
A. ReplicaSet hỗ trợ Set-based selectors (ví dụ: `environment in (production, qa)`), còn ReplicationController chỉ hỗ trợ Equality-based.
B. ReplicaSet chạy nhanh hơn.
C. ReplicaSet không hỗ trợ Rolling Update.
D. ReplicationController chỉ dùng cho Stateless App.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: A**
> **Giải thích:** ReplicaSet là thế hệ sau, linh hoạt hơn trong việc chọn (select) các Pod để quản lý nhờ Set-based selector.

### Câu 15
Khi bạn thực hiện cập nhật Image cho một **Deployment**, K8s thực hiện chiến lược **Rolling Update** như thế nào để đảm bảo Zero-downtime?
A. Tắt tất cả Pod cũ, sau đó bật tất cả Pod mới.
B. Tạo thêm một ReplicaSet mới, tăng dần số Pod mới (scale up) và giảm dần số Pod cũ (scale down) trong ReplicaSet cũ.
C. Cập nhật trực tiếp vào Container đang chạy (Hot patch).
D. Chuyển hướng traffic sang một Service khác.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Deployment quản lý 2 ReplicaSet tại thời điểm update. Nó đảm bảo luôn có một số lượng Pod nhất định sẵn sàng phục vụ user trong quá trình chuyển giao.

### Câu 16
Probe nào được sử dụng để K8s biết khi nào Pod đã **sẵn sàng nhận traffic** (nếu chưa sẵn sàng, Service sẽ không chuyển traffic vào)?
A. Liveness Probe.
B. Readiness Probe.
C. Startup Probe.
D. Ping Probe.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:**
> * **Liveness:** Container còn sống không? (Nếu không -> Restart).
> * **Readiness:** Container đã khởi động xong app chưa? (Nếu chưa -> Cô lập khỏi Service/Load Balancer, KHÔNG restart).

### Câu 17
Điều gì xảy ra nếu bạn xóa một Pod được quản lý bởi một Deployment bằng tay (lệnh `kubectl delete pod`)?
A. Pod biến mất vĩnh viễn và số lượng replica giảm đi 1.
B. Deployment (thông qua ReplicaSet) sẽ phát hiện thiếu hụt và lập tức tạo ra một Pod mới để thay thế.
C. Deployment sẽ bị lỗi.
D. Toàn bộ Deployment bị xóa theo.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Tính năng **Self-healing**. Deployment cam kết duy trì số lượng `replicas` như đã khai báo.

### Câu 18
**DaemonSet** đảm bảo điều gì?
A. Có đúng 1 bản sao của Pod chạy trên toàn bộ Cluster.
B. Có một bản sao của Pod chạy trên **mỗi** Node (hoặc một tập hợp Node được chọn).
C. Pod sẽ chạy dưới quyền root (Daemon).
D. Pod sẽ không bao giờ bị xóa.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** DaemonSet thường dùng cho các tác vụ hệ thống chạy nền trên mọi máy: Log collector (Fluentd), Monitoring agent (Node Exporter), Network proxy (kube-proxy).

### Câu 19
Để chạy một công việc chỉ diễn ra một lần rồi kết thúc (ví dụ: backup database, tính toán batch job), bạn dùng Resource nào?
A. Deployment.
B. StatefulSet.
C. Job.
D. CronJob.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** **Job** tạo Pod để chạy task, khi task hoàn thành (exit code 0), Job được coi là Complete và Pod không bị restart lại (khác với Deployment luôn restart nếu process dừng).

### Câu 20
**Static Pod** là gì?
A. Pod có IP tĩnh.
B. Pod được quản lý trực tiếp bởi Kubelet (thông qua file config trên Node), không qua API Server.
C. Pod không thể bị xóa.
D. Pod chứa nội dung website tĩnh.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Static Pod thường dùng để chạy chính các thành phần của Control Plane (như etcd, apiserver, scheduler) khi chúng được deploy dưới dạng container. Kubelet tự đọc file yaml trong thư mục `/etc/kubernetes/manifests` và chạy pod.

---

## Phần 3: Networking & Service

### Câu 21
Mỗi Pod trong Kubernetes có một địa chỉ IP riêng. Địa chỉ IP này thuộc về mạng nào?
A. Mạng của Docker Host (Bridge Network).
B. Mạng Overlay phẳng (Flat Overlay Network) bao trùm toàn bộ Cluster.
C. Mạng Internet công cộng.
D. Mạng NAT của Node.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** K8s yêu cầu mô hình mạng phẳng: Pod ở Node A giao tiếp với Pod ở Node B mà không cần NAT. Công nghệ thường dùng là VXLAN, IP-in-IP (CNI plugins như Flannel, Calico).

### Câu 22
**Service** giải quyết vấn đề gì cốt lõi của Pod?
A. Pod khởi động chậm.
B. Pod có tính chất phù du (Ephemeral), IP thay đổi mỗi khi tái tạo. Service cung cấp một IP ổn định (VIP) để truy cập nhóm Pod.
C. Pod không thể ra Internet.
D. Pod bảo mật kém.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Service là lớp trừu tượng (Abstraction layer). Client chỉ cần gọi IP của Service, Service sẽ tự load balance xuống các Pod IP thực tế (Endpoints) ở phía sau.

### Câu 23
Loại Service nào giúp phơi bày ứng dụng ra ngoài Internet thông qua một IP Public được cấp bởi Cloud Provider?
A. ClusterIP.
B. NodePort.
C. LoadBalancer.
D. ExternalName.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** `type: LoadBalancer` kích hoạt Cloud Controller Manager để yêu cầu AWS/GCP/Azure cấp một Load Balancer vật lý trỏ vào các NodePort của Cluster.

### Câu 24
**Ingress** khác gì với **Service type LoadBalancer**?
A. Ingress hoạt động ở Layer 7 (HTTP/HTTPS), cho phép định tuyến dựa trên domain/path (Host/Path-based routing), trong khi LoadBalancer thường là Layer 4.
B. Ingress rẻ hơn nhưng chậm hơn.
C. Ingress chỉ dùng cho nội bộ.
D. Ingress là một loại tường lửa.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: A**
> **Giải thích:** Nếu dùng LoadBalancer cho mỗi service, bạn tốn rất nhiều tiền (mỗi service 1 IP Public). Ingress cho phép dùng 1 LoadBalancer duy nhất (1 IP) để định tuyến cho hàng chục service khác nhau dựa trên tên miền (`api.com` vào Service A, `web.com` vào Service B).

### Câu 25
Thành phần nào thực hiện việc chuyển tiếp gói tin từ Virtual IP của Service đến IP thật của Pod (Service Proxy)?
A. CoreDNS.
B. Kubelet.
C. kube-proxy.
D. Ingress Controller.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** `kube-proxy` chạy trên mỗi Node, cài đặt các quy tắc mạng (iptables hoặc IPVS) để bắt các gói tin gửi đến ClusterIP và DNAT (Destination NAT) chúng về IP của Pod.

### Câu 26
Làm thế nào Service biết được Pod nào thuộc về nó để chuyển traffic?
A. Dựa trên tên Pod.
B. Dựa trên IP của Pod.
C. Dựa trên **Labels** và **Selectors**.
D. Dựa trên Node chứa Pod.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Service có `selector`, Pod có `label`. Service sẽ quét tất cả Pod trong namespace, cái nào khớp label thì đưa vào danh sách Endpoints.

### Câu 27
**Headless Service** (ClusterIP: None) thường được dùng để làm gì?
A. Để tắt Service.
B. Để Service Discovery trả về trực tiếp danh sách IP của các Pod thay vì 1 VIP duy nhất. Thường dùng cho StatefulSet hoặc Database Cluster tự quản lý replication.
C. Để tiết kiệm IP.
D. Để bảo mật Service.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Khi DNS query tên Headless Service, nó trả về A Record chứa list IP của Pod. Client (như DB replica) cần biết chính xác địa chỉ của các peer để sync dữ liệu, không muốn đi qua Load Balancer.

### Câu 28
CoreDNS trong Kubernetes đóng vai trò gì?
A. Cân bằng tải.
B. Service Discovery nội bộ (Chuyển đổi tên Service thành IP).
C. Quản lý DHCP.
D. Lưu trữ cấu hình mạng.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Cho phép các Pod gọi nhau bằng tên (VD: `http://my-backend`) thay vì nhớ IP.

### Câu 29
`NodePort` Service mở port trong dải nào theo mặc định?
A. 0 - 1024.
B. 30000 - 32767.
C. 80 - 443.
D. Bất kỳ port nào còn trống.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đây là dải port an toàn mặc định được quy định để tránh xung đột với các port hệ thống.

### Câu 30
Khi một Pod cố gắng truy cập Internet (Outbound), địa chỉ IP nguồn (Source IP) của gói tin khi ra khỏi Node sẽ là gì?
A. IP của Pod.
B. IP của Service.
C. IP của Node (thông qua Masquerading/SNAT).
D. IP 127.0.0.1.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** IP của Pod là IP nội bộ ảo, Router Internet không biết nó là ai. Do đó, traffic đi ra ngoài Node phải được SNAT (Source NAT) thành IP của Node vật lý.

---

## Phần 4: Storage & Stateful Applications

### Câu 31
Sự khác biệt chính giữa **StatefulSet** và **Deployment** là gì?
A. StatefulSet đảm bảo định danh cố định (Sticky Identity) cho Pod (vd: pod-0, pod-1) và thứ tự khởi động/tắt tuần tự. Deployment thì các Pod ngẫu nhiên và thay thế lẫn nhau (Interchangeable).
B. StatefulSet chỉ dùng cho database, Deployment chỉ dùng cho web.
C. StatefulSet không hỗ trợ Rolling Update.
D. Deployment dùng PVC, StatefulSet dùng HostPath.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: A**
> **Giải thích:** StatefulSet dùng cho ứng dụng cần trạng thái bền vững. Nếu `pod-0` chết, K8s tạo lại `pod-0` (cùng tên, cùng volume cũ). Deployment tạo `pod-xyz` mới tinh.

### Câu 32
**PersistentVolume (PV)** và **PersistentVolumeClaim (PVC)** tách biệt trách nhiệm như thế nào?
A. Dev tạo PV, Ops tạo PVC.
B. PV là tài nguyên vật lý thực tế (do Admin quản lý), PVC là yêu cầu xin cấp phát tài nguyên (do Developer tạo).
C. PV là ổ cứng local, PVC là ổ cứng mạng.
D. PV là tạm thời, PVC là vĩnh viễn.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Sự tách biệt này giúp Developer không cần quan tâm bên dưới là ổ cứng AWS EBS hay NFS, họ chỉ cần tạo PVC "Tôi cần 10GB". K8s sẽ tìm PV phù hợp để bind vào.

### Câu 33
Access Mode `ReadWriteOnce` (RWO) của Volume có nghĩa là gì?
A. Volume chỉ có thể đọc, không thể ghi.
B. Volume có thể được mount bởi nhiều Node cùng lúc ở chế độ đọc/ghi.
C. Volume chỉ có thể được mount ở chế độ đọc/ghi bởi **một Node duy nhất** tại một thời điểm.
D. Volume tự hủy sau 1 lần đọc/ghi.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Rất quan trọng. Block Storage (như AWS EBS) thường là RWO (chỉ gắn vào 1 máy). Nếu muốn nhiều Pod trên nhiều Node cùng ghi (RWX), phải dùng File Storage (NFS/EFS).

### Câu 34
StorageClass dùng để làm gì?
A. Phân loại dữ liệu quan trọng.
B. Hỗ trợ **Dynamic Provisioning** (Cấp phát động). Tự động tạo PV khi có PVC yêu cầu mà không cần Admin tạo tay.
C. Backup dữ liệu.
D. Mã hóa dữ liệu.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Khi Dev tạo PVC yêu cầu `storageClassName: standard`, StorageClass này sẽ gọi xuống Cloud Provider để tạo ổ đĩa thật và tạo PV tương ứng.

### Câu 35
ConfigMap và Secret khác nhau điểm nào cơ bản nhất?
A. ConfigMap lưu file, Secret lưu text.
B. Secret được mã hóa (base64 mặc định, hoặc encryption at rest), dùng cho dữ liệu nhạy cảm. ConfigMap dùng cho cấu hình thông thường (biến môi trường, file config).
C. Secret chỉ dùng cho Master Node.
D. ConfigMap không thể mount thành volume.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Về mặt kỹ thuật chúng khá giống nhau, nhưng mục đích sử dụng và cơ chế bảo mật (RBAC/Encryption) của Secret chặt chẽ hơn.

---

## Phần 5: So sánh, Bảo mật & Nâng cao

### Câu 36
So sánh **Docker Swarm** và **Kubernetes**:
A. Swarm phức tạp hơn K8s.
B. Swarm theo mô hình Imperative, K8s thuần Declarative. Swarm dễ setup nhưng thiếu tính năng mở rộng (Auto-scaling, Custom Resource) mạnh như K8s.
C. Swarm không có khái niệm Service.
D. K8s không chạy được trên máy cá nhân.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Swarm tích hợp sẵn trong Docker, dễ dùng cho quy mô nhỏ. K8s là một hệ sinh thái khổng lồ, khó học hơn nhưng mạnh mẽ và là chuẩn công nghiệp.

### Câu 37
**Taint** và **Toleration** hoạt động như thế nào?
A. Taint được gán cho Pod, Toleration gán cho Node.
B. Taint được gán cho Node để "xua đuổi" Pod. Pod chỉ được vào Node đó nếu nó có Toleration "chấp nhận" cái Taint đó.
C. Dùng để kết nối 2 cluster.
D. Dùng để ưu tiên Pod.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Ví dụ: Master Node thường có Taint `NoSchedule` để ngăn Pod ứng dụng chạy lên đó. Chỉ các System Pod có Toleration mới chạy được trên Master.

### Câu 38
**Node Affinity** khác gì với **Node Selector**?
A. Node Affinity là phiên bản mở rộng, linh hoạt hơn của Node Selector (hỗ trợ toán tử In, NotIn, Exists...).
B. Node Affinity là bắt buộc, Node Selector là tùy chọn.
C. Node Selector chạy ở Runtime, Affinity chạy ở Buildtime.
D. Chúng là một.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: A**
> **Giải thích:** Node Selector rất đơn giản (key=value). Affinity cho phép các quy tắc mềm dẻo (PreferredDuringScheduling - "thích thì vào, không thì thôi" hoặc Required - "bắt buộc").

### Câu 39
**HPA (Horizontal Pod Autoscaler)** dựa vào đâu để scale số lượng Pod?
A. Số lượng Node trong cluster.
B. Metrics (CPU/RAM usage) thu thập từ Metrics Server hoặc Custom Metrics.
C. Thời gian trong ngày.
D. Số lượng Image trong Registry.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** HPA theo dõi metrics của Pod. Nếu CPU trung bình vượt quá 80% (ngưỡng cài đặt), nó cập nhật Deployment để tăng số lượng Replica.

### Câu 40
Điều gì xảy ra khi một Pod bị OOMKilled (Out Of Memory)?
A. Pod bị xóa vĩnh viễn.
B. Container trong Pod bị kill vì dùng quá giới hạn RAM (Limit) cho phép, và K8s sẽ restart lại nó (nếu RestartPolicy cho phép).
C. Toàn bộ Node bị treo.
D. K8s tự động cấp thêm RAM cho Pod.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đây là cơ chế bảo vệ của Linux Kernel (OOM Killer) kết hợp với Cgroups. K8s chỉ nhận biết sự kiện và xử lý theo chính sách restart.

### Câu 41
**CRD (Custom Resource Definition)** là gì và tại sao nó làm nên sức mạnh của K8s?
A. Là một loại Database mới.
B. Cho phép người dùng mở rộng API của K8s, định nghĩa các loại tài nguyên mới (ví dụ: `MySQLCluster`, `PrometheusRule`) và viết Controller riêng (Operator pattern) để quản lý chúng.
C. Là công cụ debug.
D. Là chuẩn đóng gói Container.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** CRD biến K
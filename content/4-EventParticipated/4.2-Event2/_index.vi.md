---
title: "Event 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Bài thu hoạch “AWS First Cloud AI Journey Community Day - June 2026”

### Thông tin chung về sự kiện
- **Tên sự kiện:** AWS First Cloud AI Journey Community Day - June 2026
- **Đơn vị tổ chức:** AWS Study Group
- **Thời gian:** 09:00 – 12:00 | Thứ Bảy, 27/06/2026
- **Địa điểm:** Trực tuyến qua YouTube (Sự kiện gốc tại Bitexco Financial Tower, TP.HCM)
- **Vai trò:** Người tham dự

### Mục Đích Của Sự Kiện
- Chia sẻ những kinh nghiệm thực chiến từ các chuyên gia, Founder về công nghệ Cloud và AI.
- Làm rõ bài toán xây dựng và vận hành Voice Agent trên nền tảng AWS (sử dụng Amazon Bedrock).
- Giới thiệu cách ứng dụng AI (DevOps Agent, Amazon Q) để giải quyết các bài toán vận hành hệ thống và quản trị nhân sự (HR).
- Hướng dẫn thiết lập bảo mật riêng tư và tích hợp hệ thống qua MCP Server.

### Danh Sách Diễn Giả
- **Anh Steve Trần** - Founder của Cloud Thinker.
- **Anh Nghị (Renova Cloud), Anh Kiệt (Student Video Group), Anh Trung (Founder R AI)** - Các diễn giả phiên Voice AI.
- **Chị Bảo & Anh Nguyên** - Chuyên gia từ Cloud Kinetis.
- **Anh Trường & Chị Minh Anh** - Chuyên gia từ Noventis.
- **Anh Toàn Nguyễn & Anh Nghị** - Chuyên gia phiên Bảo mật & Tích hợp Amazon Q.

### Nội Dung Nổi Bật

#### Khởi nghiệp & Tư duy Cloud Thinker
- **Trọng tâm:** Giải bài toán thực tế của khách hàng.
- Anh Steve Trần chia sẻ về hành trình khởi nghiệp và bài toán vận hành Contact Center. Bài học cốt lõi được đưa ra là: Khi xây dựng bất kỳ giải pháp công nghệ nào, kỹ sư cần phải thấu hiểu sâu sắc "nỗi đau" (pain points) thực tế của khách hàng chứ không chỉ tập trung vào công nghệ.

#### Kiến trúc và Thực tiễn triển khai Voice AI
- **Trọng tâm:** Xây dựng Voice Agent cho thị trường Việt Nam.
- Anh Nghị giới thiệu chi tiết về các thành phần kiến trúc để cấu thành nên một hệ thống Voice AI hoàn chỉnh.
- Anh Kiệt trình diễn một bản demo trực tiếp và sinh động về Voice Agent được xây dựng trên lõi Amazon Bedrock.
- Anh Trung chia sẻ góc nhìn thực chiến khi triển khai cho các tập đoàn lớn tại Việt Nam: cách xử lý thách thức về nhận diện giọng vùng miền, ngôn ngữ đặc thù, và đặc biệt là kịch bản chuyển tiếp thông minh giữa AI và nhân viên con người (Human fallback).

#### Ứng dụng DevOps Agent trong vận hành
- **Trọng tâm:** Giảm thời gian xử lý sự cố hệ thống (MTTR).
- Chị Bảo và anh Nguyên demo cách sử dụng DevOps Agent để hỗ trợ đội ngũ IT điều tra nguyên nhân gốc rễ (root cause) của sự cố một cách tự động, giúp tiết kiệm đáng kể thời gian downtime của hệ thống.

#### AI và Nguồn nhân lực (HR)
- **Trọng tâm:** Chuyển đổi số nghiệp vụ Back-office.
- Anh Trường và chị Minh Anh thảo luận về cách ứng dụng Amazon Q vào lĩnh vực nhân sự. AI giúp tự động hóa quy trình tuyển dụng (sàng lọc ứng viên) và đánh giá dữ liệu nhân sự một cách khách quan, hiệu quả.

#### Bảo mật và Tích hợp với MCP Server
- **Trọng tâm:** Private Security & Model Context Protocol.
- Anh Toàn Nguyễn và anh Nghị đi sâu vào kỹ thuật thiết lập bảo mật riêng tư (private security) cho Amazon Q. Đồng thời, giải quyết bài toán kết nối AI với các hệ thống bên thứ ba (Third-party systems) thông qua MCP Server mà không làm ảnh hưởng đến an toàn dữ liệu.

### Những Gì Học Được

#### Tư Duy Sản Phẩm & Khách Hàng
- Hiểu được rằng một sản phẩm AI thành công (như Voice Agent trong Contact Center) bắt buộc phải giải quyết triệt để các rào cản thực tế (như giọng địa phương, tiếng lóng) và luôn phải có cơ chế bàn giao cho con người (human-in-the-loop) để đảm bảo chất lượng dịch vụ.

#### Tư Duy Kỹ Thuật (Technical Mindset)
- **Amazon Bedrock & LLMs:** Nắm được khái niệm cơ bản và luồng hoạt động khi xây dựng các tác vụ GenAI trên AWS.
- **MCP Server:** Lần đầu tiên tiếp cận với kiến trúc MCP giúp mở rộng khả năng của AI Model một cách an toàn.
- **DevOps Automation:** Nhận ra rằng AI không chỉ dùng để chat, mà còn có khả năng đọc log, phân tích root cause và hỗ trợ engineer sửa lỗi hệ thống (DevOps Agent).

### Ứng Dụng Vào Công Việc

- **Phân tích vấn đề từ gốc rễ:** Thay vì vội vàng lao vào code, tôi sẽ áp dụng tư duy "Cloud Thinker" – phân tích kỹ bài toán và bối cảnh trước khi thiết kế giải pháp cho dự án thực tập.
- **Tích hợp Amazon Bedrock:** Đưa Amazon Bedrock vào danh sách các dịch vụ cần tìm hiểu sâu (research) để phục vụ cho các dự án tích hợp GenAI trong tương lai.
- **Quy trình xử lý sự cố:** Học hỏi luồng phân tích root cause từ phiên DevOps Agent để cải thiện kỹ năng debug và fix bug trong quá trình làm lab AWS.

### Trải nghiệm trong event

Dù chỉ tham dự trực tuyến qua YouTube, nhưng **FCAJ Community Day** đã mang lại cho tôi những trải nghiệm vô cùng sống động:

#### Demo thực chiến, dễ hiểu
- Việc anh Kiệt demo trực tiếp Voice Agent trên nền Amazon Bedrock hay chị Bảo demo DevOps Agent giúp các kiến thức lý thuyết trở nên cực kỳ trực quan và dễ hình dung.

#### Góc nhìn đa chiều từ các chuyên gia
- Sự kiện có sự góp mặt của các Founder và chuyên gia từ nhiều công ty khác nhau (Cloud Thinker, Renova Cloud, R AI, Cloud Kinetis, Noventis). Nhờ đó, tôi nhận được những góc nhìn rất đa dạng: từ bài toán kỹ thuật (bảo mật, kiến trúc) cho đến bài toán kinh doanh (vận hành HR, Contact Center).

#### Tinh thần cộng đồng
- Phần kết thúc sự kiện với bức ảnh lưu niệm và các hoạt động giveaway cho thấy sự gắn kết và năng lượng rất lớn từ cộng đồng AWS Study Group, tạo cho tôi rất nhiều động lực để tiếp tục theo đuổi con đường công nghệ đám mây.

#### Bài học rút ra
- Trong kỷ nguyên GenAI, việc làm chủ công nghệ lõi (như Bedrock) là cần thiết, nhưng biết cách tích hợp nó một cách an toàn (MCP Server, Private Security) và giải quyết đúng bài toán của khách hàng mới là điều làm nên giá trị của một kỹ sư.
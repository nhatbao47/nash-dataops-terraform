# DataOps Presentation Guide

This document is a speaker-ready guide for a DataOps presentation. Sections 1
through 4 explain the general concepts. Section 5 connects those concepts to
the AWS project implemented in this repository.

## Table of Contents

1. [Overview DataOps](#1-overview-dataops)
2. [Overview Data Pipeline](#2-overview-data-pipeline)
3. [Data Pipeline Development & Management](#3-data-pipeline-development--management)
4. [Implementation DataOps in AWS](#4-implementation-dataops-in-aws)

## 1. Overview DataOps

### Data Product

A data product is a trusted, reusable data asset designed for a specific group
of consumers and a specific decision-making purpose.

A good data product has clear ownership, documented meaning, quality
expectations, freshness expectations, and a known audience.

### Speaker Notes: Data Product

Say:

> Trước khi nói về DataOps, chúng ta cần hiểu khái niệm data product. Data
> product là một sản phẩm dữ liệu được thiết kế để người khác sử dụng lại cho
> phân tích, báo cáo hoặc ra quyết định.

> Data product không chỉ là một file hoặc một bảng dữ liệu. Nó cần có người sở
> hữu, ý nghĩa rõ ràng, quy tắc chất lượng, kỳ vọng về độ mới của dữ liệu.
### Example Data Product: NYC TLC Trip Record Data

Reference dataset:
`https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page`

### Speaker Notes: What Can A Data Team Do With TLC Trip Data?

Say:

> Ở đây em có dữ liệu thô của một cơ quan chuyên thu thập dữ liệu của taxi truyền thống, taxi công nghệ và các dịch vụ tương tự tại New York. Dữ liệu này được publish hằng tháng. Vậy chúng ta có thể làm gì với dữ liệu này để tạo thành data product.

Example 1: demand and operations analysis.

> Trường hợp đầu tiên là phân tích nhu cầu di chuyển. Data team có thể dùng
> dữ liệu để trả lời các câu hỏi như: giờ nào
> trong ngày có nhu cầu cao nhất, khu vực nào có nhiều người đi và xuống,
> ngày thường và cuối tuần khác nhau như thế nào ...

> Kết quả của phân tích này có thể hỗ trợ đội vận hành ra quyết định: cần phân
> bổ xe ở khu vực nào, thời điểm nào dễ thiếu cung, tuyến đường nào có nhu cầu
> cao, hoặc khu vực nào có dấu hiệu thay đổi hành vi di chuyển.

> Ngoài ra, chính phủ cũng có thể sử dụng dữ liệu để ra quyết định về quy hoạch
> giao thông, hạ tầng, 

Example 2: revenue and pricing analysis.

> Trường hợp thứ hai là phân tích doanh thu và chi phí chuyến đi. Vì dataset có
> thông tin về các loại phí, data team có thể
> xây metric như doanh thu theo khu vực, doanh thu theo khung giờ, giá trị
> trung bình mỗi chuyến ... 

> Những dữ liệu này giúp business user hiểu sâu hơn về "giá trị kinh tế của các chuyến đi nằm ở đâu".

> Chúng ta đã hình dung được một sản phẩm data trông như thế nào. Vậy câu hỏi tiếp theo là DataOps sẽ hỗ  trợ phát triển sản phẩm đó như thế nào và nó khác gì so với cách làm truyền thống 

### Key Message

DataOps is the discipline of applying software engineering, automation,
quality control, collaboration, and operational monitoring to data delivery.
The goal is not only to move data, but to deliver trusted data products
quickly, repeatedly, and safely.

In simple terms:

> DataOps is DevOps thinking applied to data pipelines, data platforms, data
> quality, and analytics delivery.

### Speaker Notes: Key Message

Say:

> Thông điệp chính ở đây là DataOps không chỉ là một công cụ hay vị trí công việc. DataOps là cách tổ chức việc phát triển và vận hành dữ liệu để data
> product được tạo ra nhanh hơn, ổn định hơn và đáng tin cậy hơn.

> Nếu DevOps giúp software team release phần mềm một cách có kiểm soát, thì
> DataOps áp dụng tư duy tương tự cho dữ liệu. Chúng ta cần có version control,
> kiểm thử, tự động hóa, monitoring, ownership và quy trình xử lý lỗi và được áp dụng cho vòng đời phát triển và vận hành data product.

### What Problem DataOps Solves

Many data teams start with a simple goal: move data from source systems into a
report, data warehouse, lakehouse, or machine learning feature set. Over time,
that simple goal becomes difficult because data systems change constantly:

- Source schemas change without warning.
- Files arrive late, duplicated, incomplete, or malformed.
- Transformation code grows into many separate scripts.
- Business logic is copied across dashboards and jobs.
- Data quality issues are discovered by end users instead of the data team.
- Infrastructure is manually configured and hard to reproduce.

Using NYC TLC Trip Record Data as an example, the work may sound simple at
first: download monthly Parquet files, combine trips, join the taxi zone lookup,
and build dashboards. In practice, many operational problems appear:

- Monthly files may arrive later than expected or be reprocessed.
- Different trip types may have different fields and business meanings.
- A pickup or dropoff location ID may not match the zone lookup.
- Trip duration can be invalid if timestamps are missing or reversed.
- Fare and fee fields may need consistent definitions before revenue analysis.
- Analysts may calculate peak-hour demand differently in different dashboards.
- A dashboard may show a changed number without clear lineage back to the
  source file and transformation logic.

DataOps addresses these problems by treating data delivery as a managed
software and operations process.

### Speaker Notes: What Problem DataOps Solves

Say:

> Bây giờ quay lại câu hỏi: vì sao cần DataOps? Vấn đề bắt đầu từ một thực tế
> rất quen thuộc. Ban đầu pipeline thường khá đơn giản, nhưng theo thời gian nó
> trở nên khó kiểm soát. Nguồn dữ liệu thay đổi, file đến trễ, schema thay đổi,
> logic xử lý bị copy ở nhiều nơi, và cuối cùng người dùng dashboard là người
> đầu tiên phát hiện số liệu sai.

Use the TLC example:

> Lấy ví dụ với TLC Trip Record Data. Nghe qua thì bài toán có vẻ đơn giản:
> download file thô theo tháng rồi phân tích và tạo ra dashboard
> về số chuyến, khu vực, tuyến đường và doanh thu. Nhưng khi triển khai thật, data team sẽ gặp nhiều vấn đề.

> File tháng mới có thể đến trễ hoặc được publish lại. Một số record có thể thiếu trường dữ liệu. Có record có trường dữ liệu không chính xác. Ví dụ: thời gian
> trả khách lại trước thời gian đón khách.

> Nếu không có DataOps, những lỗi này có thể đi thẳng vào dashboard. Khi đó
> business user chỉ nhìn thấy số liệu sai và đưa ra quyết định sai.

> Khi business user hỏi tại sao, data team mới bắt đầu kiểm tra job, file, query và logic. Cách làm này tốn thời gian và làm giảm niềm tin vào dữ liệu.

> DataOps giúp chuyển cách làm từ bị động sang chủ động. Thay vì chỉ hỏi "job
> có chạy xong không?", DataOps hỏi thêm: dữ liệu có đủ không, có đúng schema
> không, có đạt quality rule không, có trace được nguồn không, và nếu lỗi xảy
> ra thì có thể phục hồi hoặc rerun được không.

> Vì vậy, DataOps không chỉ giúp pipeline chạy tự động hơn. Nó giúp data team
> kiểm soát rủi ro, giảm lỗi đến người dùng cuối, và tăng niềm tin vào dữ liệu
> được dùng cho quyết định.

Transition:

> Sau khi hiểu vấn đề, chúng ta có thể nhìn vào các mục tiêu cụ thể mà DataOps
> hướng tới.

### DataOps Objectives

| Objective | What it means |
| --- | --- |
| Speed | Reduce the time from raw data to usable insight |
| Reliability | Make pipeline runs predictable and recoverable |
| Quality | Detect invalid, incomplete, or unexpected data before consumers use it |
| Reproducibility | Make code, infrastructure, and configuration repeatable |
| Observability | Know what ran, what failed, what changed, and what data was produced |
| Collaboration | Align data engineers, analysts, platform teams, and business users |
| Governance | Make ownership, lineage, access, and quality rules explicit |

### Speaker Notes: DataOps Objectives

Say:

> Các mục tiêu của DataOps có thể được hiểu như một bộ tiêu chí để đánh giá một
> data platform hoạt động tốt đến đâu

Then walk through the objectives:

> Speed nghĩa là rút ngắn thời gian từ dữ liệu thô đến insight. Nhưng tốc độ
> không có nghĩa là bỏ qua kiểm soát. DataOps muốn pipeline nhanh hơn nhưng vẫn
> an toàn hơn.

> Reliability nghĩa là pipeline chạy ổn định, có thể dự đoán được, và khi lỗi
> xảy ra thì team biết lỗi nằm ở đâu. Một pipeline đáng tin cậy không chỉ là
> pipeline ít fail, mà còn là pipeline fail rõ ràng và dễ phục hồi.

> Quality nghĩa là dữ liệu phải được kiểm tra trước khi người dùng cuối sử dụng.
> Nếu dữ liệu thiếu, sai kiểu, sai logic hoặc bất thường, pipeline nên phát hiện
> sớm thay vì để lỗi xuất hiện trên dashboard.

> Reproducibility nghĩa là cùng một code, cùng một cấu hình, cùng một input thì
> phải có thể tạo lại output giống nhau. Đây là điều rất quan trọng khi cần
> debug, backfill hoặc giải thích vì sao số liệu thay đổi.

> Observability giúp team biết pipeline đã chạy gì, tạo ra dữ liệu gì, dữ liệu
> có mới không, có đủ không, và bước nào đang có vấn đề.

> Collaboration và Governance nhắc chúng ta rằng dữ liệu không chỉ là chuyện
> kỹ thuật. Data engineer, analyst, platform team và business user cần cùng
> hiểu metric, ownership, access, lineage và quality expectation.

Close with:

> Nếu tóm gọn, DataOps cố gắng cân bằng ba yếu tố: nhanh hơn, đáng tin cậy hơn,
> và dễ kiểm soát hơn.

Transition:

> Các mục tiêu này cần được biến thành nguyên tắc làm việc hằng ngày. Đó là lý
> do chúng ta cần các DataOps principles.

### Core DataOps Principles

| Principle | Explanation |
| --- | --- |
| Automation | Automated data pipelines, testing, and deployment |
| Collaboration | Breaking down silos between data teams, IT, and business users |
| Continuous Integration / Delivery | Frequent, reliable data releases |
| Monitoring & Observability | Real-time data quality and pipeline monitoring |
| Version Control | Tracking changes to data, code, and configurations |

### Speaker Notes: Core DataOps Principles

Say:

> Các nguyên tắc DataOps là cách biến mục tiêu thành hành động cụ thể trong
> team.

Then explain:

> "Automation" nghĩa là những bước lặp lại như ingestion, transformation,
> testing và deployment nên được tự động hóa. Nếu pipeline phụ thuộc quá nhiều
> vào thao tác thủ công, rủi ro lỗi vận hành sẽ rất cao.

> "Collaboration" nghĩa là phá vỡ khoảng cách giữa các bên. Data engineer có thể xây pipeline, nhưng business user mới là người xác
> nhận dữ liệu có trả lời đúng câu hỏi thực tế hay không.

> "Continuous Integration / Delivery" nghĩa là thay đổi trong pipeline, SQL,
> quality rule hoặc dashboard nên được tích hợp, kiểm thử và release thường
> xuyên theo cách đáng tin cậy. Mục tiêu là release dữ liệu nhanh hơn nhưng vẫn
> kiểm soát được rủi ro.

> Khi nói về "Monitoring & Observability" trong DataOps, hãy tập trung vào 4 câu hỏi rất
> thực tế. Pipeline có chạy thành công không? Dữ liệu có đến đúng thời gian
> không? Dữ liệu có đầy đủ, hợp lệ và mới không? Và cuối cùng, người dùng có
> thể tin và giải thích metric không? Khi bạn thiết kế được hệ thống mà trả lời được 4 câu hỏi này thì có nghĩa là bạn đã đáp ứng được nguyên tắc "Monitoring & Observability" trong DataOps.

> "Version Control" nghĩa là mọi thay đổi quan trọng đối với dữ liệu, code và
> cấu hình cần được tracking. Khi số liệu thay đổi, team có thể truy lại thay
> đổi nào đã xảy ra, ai thay đổi và vì sao.

Transition:

> Khi có các nguyên tắc này, DataOps trở nên khác với cách làm ETL truyền thống.
> Phần tiếp theo giúp so sánh sự khác biệt đó.

### DataOps vs Traditional ETL

| Traditional ETL | DataOps |
| --- | --- |
| Job-centric | Product-centric |
| Manual deployment | Automated deployment |
| Reactive data quality | Proactive data contracts and quality gates |
| Limited lineage | Traceable source, transformation, and output |
| Siloed engineering and analytics | Shared ownership across teams |
| Hard to reproduce environments | Infrastructure and configuration as code |
| Monitoring focused on job failures | Monitoring includes freshness, completeness, validity, and business impact |

### Speaker Notes: DataOps vs Traditional ETL

Say:

> Phát triển hệ thống dữ liệu truyền thống thường tập trung vào câu hỏi: làm sao extract, transform và
> load dữ liệu từ A sang B. DataOps mở rộng câu hỏi đó: làm sao để toàn bộ quá
> trình delivery dữ liệu có thể được version, test, deploy, monitor và cải tiến
> liên tục.

### Who Participates in DataOps

| Role | Responsibility |
| --- | --- |
| Data Engineer | Builds and maintains ingestion, transformation, and quality logic |
| Analytics Engineer | Models business-ready tables and metrics |
| Data Analyst | Validates business meaning and dashboard usefulness |
| Platform Engineer | Provides reliable infrastructure, deployment, and runtime foundations |
| Data Steward / Owner | Defines quality expectations, access, and governance rules |
| Business User | Consumes trusted data and provides feedback on value |

### Speaker Notes: Who Participates in DataOps

Say:

> Sau khi hiểu DataOps là gì, nó giải quyết vấn đề gì, mục tiêu và nguyên tắc
> của nó là gì, chúng ta quay lại câu hỏi rất thực tế: ai tham gia vào DataOps?

> DataOps không phải trách nhiệm của một người hoặc một tool. Nó là cách nhiều
> vai trò phối hợp để đưa dữ liệu từ trạng thái thô thành dữ liệu có thể tin
> cậy. Data engineer xây pipeline và quality logic. Analytics engineer hoặc BI
> engineer biến dữ liệu thành model, metric và bảng phân tích dễ dùng. Data
> analyst kiểm tra ý nghĩa kinh doanh, tìm insight và xây dashboard. Platform
> engineer hỗ trợ hạ tầng, bảo mật và vận hành. Data steward hoặc data owner
> định nghĩa rule, ownership và governance. Business user xác nhận dữ liệu có
> trả lời đúng câu hỏi thực tế hay không.

> Điểm quan trọng là data team không chỉ "lấy data rồi làm chart". Họ cùng chịu
> trách nhiệm về nguồn dữ liệu, chất lượng, logic xử lý, khả năng vận hành và
> niềm tin của người dùng cuối vào số liệu.

## 2. Overview Data Pipeline

### Key Message

A data pipeline is a controlled flow that moves data from raw sources to
trusted outputs. It usually performs ingestion, validation, transformation,
storage, and publishing.

The important idea is that a pipeline should not be a black box. Each stage
should have a clear purpose, clear inputs, clear outputs, and clear checks.

### Speaker Notes: Key Message

Say:

> Ở phần trước, chúng ta đã nói về DataOps và data product. Bây giờ câu hỏi là:
> để tạo ra một data product đáng tin cậy, dữ liệu phải đi qua những bước nào?
> Đó chính là vai trò của data pipeline.

> Data pipeline là một luồng xử lý có kiểm soát. Nó đưa dữ liệu từ nguồn ban
> đầu đi qua các bước như thu thập, lưu trữ, kiểm tra, biến đổi và xuất bản cho
> người dùng hoặc hệ thống khác sử dụng.

> Điểm quan trọng là pipeline không nên là một hộp đen. Mỗi stage cần có đầu
> vào rõ ràng, đầu ra rõ ràng, trách nhiệm rõ ràng và tiêu chí kiểm tra rõ ràng.
> Khi có lỗi, team phải biết lỗi xảy ra ở bước nào và dữ liệu nào bị ảnh hưởng.

### Common Pipeline Stages

| Layer | Location | Purpose |
| --- | --- | --- |
| Source | Application databases, APIs, files, streams, SaaS tools | Systems where data originates |
| Ingestion | Landing zone, raw storage, message queue | Captures source data with minimal change |
| Raw / Bronze | Data lake raw layer | Preserves original data for replay and audit |
| Clean / Silver | Standardized storage | Cleans types, names, deduplication, and basic validity |
| Curated / Gold | Warehouse, marts, feature store | Business-ready models, metrics, and entities |
| Consumption | BI, reverse ETL, ML, APIs | Data products used by people or systems |

### Speaker Notes: Common Pipeline Stages

Say:

> Một pipeline thường không nên đưa dữ liệu từ source thẳng vào dashboard. Nếu
> làm như vậy, chúng ta rất khó kiểm soát lỗi và rất khó giải thích dữ liệu đã
> thay đổi ở đâu.

Then explain:

> Source là nơi dữ liệu phát sinh. Ingestion là bước đưa dữ liệu vào hệ thống
> data platform với ít thay đổi nhất. Raw giữ dữ liệu thô để có thể audit
> hoặc replay. Clean là nơi dữ liệu được chuẩn hóa và làm sạch. Curated là nơi dữ
> liệu đã sẵn sàng cho business logic, dashboard hoặc machine learning.

### Batch, Streaming Pipelines

| Pattern | When it fits | Tradeoff |
| --- | --- | --- |
| Batch | Daily/hourly reporting, large historical processing | Simpler and cost-efficient, but not real time |
| Streaming | Event-driven analytics, monitoring, fraud detection | Low latency, but more operational complexity |

### Speaker Notes: Batch, Streaming Pipelines

Say:

> Không phải pipeline nào cũng cần real-time. Lựa chọn batch, streaming phụ thuộc vào câu hỏi kinh doanh cần trả lời nhanh đến mức nào và team
> có đủ khả năng vận hành độ phức tạp đó không.

> Batch phù hợp cho báo cáo hằng ngày hoặc dữ liệu lịch sử lớn. Streaming phù
> hợp khi cần phản ứng gần như tức thì, ví dụ phát hiện gian lận hoặc monitoring.

> Điểm cần nhấn mạnh là kiến trúc tốt không phải lúc nào cũng là kiến trúc phức
> tạp nhất. Kiến trúc tốt là kiến trúc đáp ứng đúng yêu cầu về latency, chi phí,
> độ tin cậy và khả năng vận hành của team.

### Common Pipeline Architecture Patterns

The following patterns are common in modern data platforms. A real platform
may combine several of them depending on latency, cost, governance, and
consumer needs.

#### Medallion Architecture

The Medallion architecture organizes data into progressive quality layers:

- Bronze: raw, replayable, minimally transformed.
- Silver: cleaned, typed, deduplicated, conformed.
- Gold: business-ready, aggregated, dimensional, or metric-ready.

This pattern is useful because it separates source preservation from business
curation.

#### ELT and ETL

| Pattern | Description | Good fit |
| --- | --- | --- |
| ETL | Transform data before loading into the target warehouse | Strict target schemas, legacy warehouses, heavy pre-processing |
| ELT | Load data first, then transform inside the data platform | Modern warehouses/lakehouses, flexible analytics, scalable compute |

Many modern platforms use a mix: raw data lands first, then transformation
jobs create curated datasets.

#### Orchestration

Pipeline orchestration controls order, dependencies, retries, branching, and
failure behavior.

Typical orchestration responsibilities:

- Start jobs in the right sequence.
- Pass runtime parameters.
- Wait for long-running work.
- Retry temporary failures.
- Branch when quality checks fail.
- Record success and failure status.
- Trigger downstream consumers only when data is ready.

#### Data Quality Gates

A data quality gate is a decision point in the pipeline. It checks whether the
data is good enough to continue.

Common quality checks:

- Row count is above a minimum threshold.
- Required fields are complete.
- Numeric values are within expected bounds.
- Dates fall within expected windows.
- Primary or business keys are unique.
- Referential integrity holds between facts and dimensions.
- Freshness is within the agreed service level.

### Speaker Notes: Common Pipeline Architecture Patterns

Say:

> Sau khi hiểu các stage cơ bản của pipeline và lựa chọn batch hay streaming,
> câu hỏi tiếp theo là: chúng ta nên tổ chức kiến trúc pipeline như thế nào để
> dễ vận hành, dễ kiểm soát chất lượng và dễ phục vụ nhiều nhóm người dùng?

> Có một vài pattern thường gặp trong data platform hiện đại. Những pattern này
> không loại trừ nhau. Trong một hệ thống thực tế, chúng ta thường kết hợp nhiều
> pattern để giải quyết các nhu cầu khác nhau về lưu trữ, xử lý, kiểm soát chất
> lượng và publishing.

Then explain Medallion Architecture:

> Pattern đầu tiên là Medallion Architecture. Ý tưởng của nó là chia dữ liệu
> thành nhiều lớp chất lượng tăng dần. Bronze giữ dữ liệu thô để có thể audit
> hoặc replay. Silver là nơi dữ liệu được làm sạch, chuẩn hóa kiểu dữ liệu,
> loại bỏ trùng lặp và chuẩn bị cho các bước xử lý tiếp theo. Gold là lớp dữ
> liệu đã sẵn sàng cho business, ví dụ dashboard, metric, báo cáo hoặc machine
> learning.

> Cách chia lớp này giúp data team không làm mất dữ liệu gốc, nhưng vẫn có thể
> tạo ra dữ liệu sạch và dễ dùng cho người dùng cuối. Khi số liệu có vấn đề,
> team có thể trace lại từ Gold về Silver và Bronze để biết lỗi phát sinh ở đâu.

Then explain ETL and ELT:

> Pattern tiếp theo là ETL và ELT. ETL nghĩa là transform dữ liệu trước rồi mới
> load vào hệ thống đích. Cách này phù hợp khi target system có schema chặt chẽ
> hoặc cần xử lý dữ liệu nhiều trước khi lưu. ELT nghĩa là load dữ liệu vào data
> platform trước, sau đó mới transform bên trong warehouse hoặc lakehouse.

> Nhiều platform hiện đại dùng kết hợp cả hai. Ví dụ dữ liệu raw được đưa vào
> data lake trước, sau đó các job transformation tạo ra dataset đã được curated
> cho phân tích.

Then explain orchestration:

> Khi pipeline có nhiều bước, chúng ta cần orchestration. Orchestration quyết
> định job nào chạy trước, job nào chạy sau, khi nào retry, khi nào dừng, khi
> nào branch sang luồng xử lý lỗi, và khi nào downstream consumer được phép sử
> dụng dữ liệu.

> Nếu không có orchestration rõ ràng, pipeline rất dễ trở thành một chuỗi script
> rời rạc. Khi lỗi xảy ra, team khó biết bước nào đã chạy, bước nào chưa chạy,
> và dữ liệu hiện tại đang ở trạng thái nào.

Then explain data quality gates:

> Cuối cùng là data quality gate. Đây là điểm kiểm soát trước khi dữ liệu đi
> tiếp sang lớp đáng tin cậy hơn hoặc trước khi publish cho dashboard. Quality
> gate có thể kiểm tra số lượng dòng, field bắt buộc, khoảng giá trị hợp lệ,
> ngày tháng, khóa duy nhất, quan hệ giữa fact và dimension, hoặc độ mới của dữ
> liệu.

> Điểm quan trọng là quality gate biến kiểm tra chất lượng thành một phần của
> pipeline. Dữ liệu không đạt yêu cầu nên bị chặn sớm, thay vì đi vào dashboard
> rồi mới để người dùng phát hiện.

Close with:

> Tóm lại, các architecture pattern này giúp pipeline có cấu trúc rõ ràng hơn:
> Medallion quản lý các lớp dữ liệu, ETL và ELT quản lý cách transform, orchestration
> quản lý thứ tự chạy, và quality gate quản lý điều kiện để dữ liệu được đi tiếp.

## 3. Data Pipeline Development & Management

### Key Message

Data pipeline development should be managed like software development, with
clear design, version control, testing, deployment, ownership, and operational
support.

### Development Lifecycle

| Stage | Activities |
| --- | --- |
| Requirement | Define source, consumer, expected freshness, quality rules, and ownership |
| Design | Choose architecture, storage layers, schema strategy, and orchestration pattern |
| Build | Implement ingestion, transformation, validation, and publishing |
| Test | Validate code, schema, business rules, data quality, and edge cases |
| Deploy | Promote changes safely across environments |
| Operate | Monitor runs, investigate failures, control cost, and improve performance |
| Improve | Add tests, refactor models, improve reliability, and respond to business feedback |

### Speaker Notes: Development Lifecycle

Say:

> Phát triển data pipeline cũng cần lifecycle giống software. Chúng ta bắt đầu
> từ requirement, sau đó design, build, test, deploy, operate và liên tục improve.

Then explain:

> Điểm khác biệt là yêu cầu của data pipeline không chỉ là “lấy dữ liệu từ A sang B”. Nó còn
> bao gồm độ mới của dữ liệu, người sở hữu dữ liệu, quy tắc kiểm tra chất lượng, kỳ vọng về
> cấu trúc dữ liệu, người sử dụng dữ liệu, và cách xử lý khi dữ liệu đến trễ hoặc bị sai.

> Nếu bỏ qua lifecycle này, pipeline dễ trở thành một tập script khó kiểm soát.
> Nếu quản lý đúng, pipeline trở thành một data product có thiết kế, kiểm thử,
> vận hành và cải tiến rõ ràng.

### Version Control Strategy

Version control should include:

- Transformation code.
- SQL models and stored procedures.
- Data quality rules.
- Orchestration definitions.
- Infrastructure definitions.
- Dashboard definitions or BI configuration where possible.
- Documentation and runbooks.

This creates reviewable history and reduces undocumented production changes.

### Speaker Notes: Version Control Strategy

Say:

> Version control giúp data team biến thay đổi dữ liệu thành thay đổi có thể
> review được. Không chỉ application code mới cần Git; pipeline code, SQL,
> quality rules, workflow, infrastructure và documentation cũng cần lịch sử rõ
> ràng.

> Khi số liệu trên dashboard thay đổi, version control giúp team truy ngược:
> có thay đổi logic nào không, schema có đổi không, quality rule có được cập
> nhật không, hoặc infrastructure có thay đổi gì ảnh hưởng đến output không.

### Environment Management

Most mature data teams separate environments:

| Environment | Purpose |
| --- | --- |
| Local | Fast development and unit-level checks |
| Dev | Integration testing with realistic services |
| Staging / UAT | Business validation and release testing |
| Production | Trusted data delivery for consumers |

The same pipeline logic should be deployable across environments with
environment-specific configuration.

### Speaker Notes: Environment Management

Say:

> Environment management giúp team thử nghiệm thay đổi mà không làm ảnh hưởng
> đến dữ liệu production. Local dùng để phát triển nhanh, dev dùng để test tích
> hợp, staging hoặc UAT dùng cho business validation, production dùng cho dữ liệu
> đáng tin cậy.

> Điều quan trọng là cùng một pipeline logic nên chạy được ở nhiều môi trường,
> chỉ khác cấu hình như bucket, database, secret hoặc schedule. Nếu mỗi môi
> trường được làm thủ công khác nhau, lỗi drift sẽ xuất hiện rất nhanh.

### Testing Strategy

Pipeline testing should include multiple levels:

| Test Type | Example |
| --- | --- |
| Unit test | Validate transformation functions or SQL logic |
| Schema test | Confirm expected columns and data types |
| Data quality test | Check completeness, uniqueness, valid ranges |
| Regression test | Compare output after code changes |
| Integration test | Run multiple pipeline stages together |
| Acceptance test | Confirm business metrics match expectations |

### Speaker Notes: Testing Strategy

Say:

> Testing trong data pipeline có nhiều tầng. Unit test kiểm tra logic nhỏ, schema
> test kiểm tra cấu trúc dữ liệu, data quality test kiểm tra dữ liệu thực tế,
> regression test kiểm tra thay đổi có làm sai output cũ không, và acceptance
> test xác nhận metric có đúng ý nghĩa kinh doanh không.

> Với dữ liệu, kiểm thử không chỉ nằm trong code. Kiểm thử còn nằm trong chính
> dữ liệu: giá trị bị thiếu, dữ liệu bị trùng lặp, giá trị nằm ngoài khoảng hợp
> lệ, mối quan hệ giữa các bảng có đúng không, dữ liệu có đủ mới không, và số
> lượng bản ghi có bất thường không. Đây là lý do DataOps nói rằng dữ liệu cũng
> cần được kiểm thử như code.

### Release Management

Pipeline changes should be released deliberately:

- Review code changes.
- Run automated checks.
- Deploy infrastructure changes safely.
- Backfill only when needed.
- Keep rollback or recovery procedures.
- Communicate metric-impacting changes to consumers.

### Speaker Notes: Release Management

Say:

> Release management giúp thay đổi pipeline một cách có kiểm soát. Một thay đổi
> nhỏ trong SQL hoặc transformation có thể làm thay đổi metric kinh doanh, vì
> vậy không nên deploy một cách âm thầm.

> Trước khi release, team nên review code, chạy test, xem plan infrastructure,
> xác định có cần backfill không, và thông báo nếu metric trên dashboard sẽ thay
> đổi. Đây là cách bảo vệ niềm tin của người dùng dữ liệu.

## 4. Implementation DataOps in AWS

This section connects the general DataOps concepts to the AWS implementation
in this repository.

### Project Context

The project implements a small but complete DataOps flow:

- Raw NYC FHVHV taxi trip data is manually uploaded to S3 Bronze.
- AWS Glue crawlers discover Bronze and Silver datasets.
- AWS Glue Spark jobs transform raw trip files into curated Silver Parquet.
- AWS Glue Data Quality evaluates data contracts before loading the warehouse.
- AWS Step Functions orchestrates the pipeline and branches on success/failure.
- Amazon Redshift stores the Gold dimensional model.
- Local Metabase connects to Redshift and provides presentation dashboards.

### Speaker Notes: Project Context

Say:

> Phần này là lúc chúng ta nối các khái niệm DataOps phía trước với demo thực
> tế trên AWS. Project này không cố gắng xây một enterprise platform đầy đủ,
> nhưng nó có đủ các thành phần quan trọng để minh họa DataOps end to end.

> Dữ liệu raw được upload vào S3 Bronze, Glue crawler tạo metadata, Glue Spark
> xử lý sang Silver, Glue Data Quality kiểm tra dữ liệu trước khi load, Step
> Functions điều phối toàn bộ flow, Redshift lưu mô hình Gold, và Metabase hiển
> thị dashboard cho người dùng phân tích.

> Điểm nên nhấn mạnh là mỗi service có một trách nhiệm riêng. S3 lưu data layer,
> Glue xử lý và catalog dữ liệu, Step Functions điều phối, Redshift phục vụ
> analytics, và Metabase là lớp consumption.

### Speaker Notes: Explaining The Current Flow Diagram

Use these Vietnamese notes when presenting `media/current-flow-architecture.png`.

Say:

> Slide này mô tả luồng DataOps hiện tại của project trên AWS. Mục tiêu của
> kiến trúc này là biến dữ liệu thô được upload thủ công thành dữ liệu sạch,
> có kiểm soát chất lượng, được lưu trong Redshift và sẵn sàng cho dashboard
> Metabase.

Then explain the diagram from left to right:

> Đầu tiên, dữ liệu đầu vào được upload thủ công vào S3 Bronze. Trong ví dụ
> này, chúng ta có hai nhóm dữ liệu chính: file Parquet chứa dữ liệu chuyến xe
> FHVHV taxi, và file CSV chứa thông tin tham chiếu về taxi zone. Bronze là
> vùng dữ liệu thô, nghĩa là dữ liệu được giữ gần với trạng thái ban đầu nhất
> để phục vụ việc trace lại nguồn nếu cần.

> Sau khi dữ liệu nằm trong Bronze, AWS Step Functions đóng vai trò điều phối
> toàn bộ pipeline. Thay vì chạy từng job thủ công, Step Functions quyết định
> thứ tự chạy crawler, job xử lý dữ liệu, bước kiểm tra chất lượng, và bước load
> vào Redshift. Điểm quan trọng ở đây là pipeline có một luồng điều khiển rõ
> ràng, có thể theo dõi được từng bước thành công hay thất bại.

> Glue Bronze Crawler sẽ đọc dữ liệu trong S3 Bronze và cập nhật metadata vào
> Glue Data Catalog. Metadata này giúp các Glue job hiểu schema của dữ liệu thô
> mà không cần hard-code toàn bộ cấu trúc file.

> Tiếp theo, Glue Spark job xử lý dữ liệu từ Bronze sang Silver. Ở bước này,
> pipeline chuẩn hóa dữ liệu, chọn các cột cần thiết, ép kiểu dữ liệu, enrich
> thêm thông tin zone, tính toán các field phục vụ phân tích, và ghi kết quả
> xuống S3 Silver dưới dạng Parquet đã được partition theo year, month và
> run_id. Silver là lớp dữ liệu đã được làm sạch và có cấu trúc tốt hơn.

> Sau khi Silver được tạo, Glue Silver Crawler tiếp tục cập nhật metadata cho
> lớp Silver. Bước này giúp các bước downstream có thể đọc dữ liệu Silver như
> một dataset có schema rõ ràng.

> Trước khi dữ liệu được đưa vào kho dữ liệu, Glue Data Quality chạy các rule
> kiểm tra chất lượng. Ví dụ: kiểm tra các field bắt buộc không bị null, kiểm
> tra thời gian chuyến đi hợp lệ, kiểm tra location id có giá trị, và kiểm tra
> các điều kiện cơ bản để đảm bảo dữ liệu đủ tin cậy cho phân tích.

> Nếu dữ liệu không đạt chất lượng, pipeline không tiếp tục load vào Redshift.
> Thay vào đó, dữ liệu lỗi được chuyển sang vùng Quarantine trên S3. Đây là
> điểm rất quan trọng trong DataOps: dữ liệu xấu không được âm thầm đi vào
> dashboard hoặc báo cáo kinh doanh. Pipeline fail có chủ đích để team biết cần
> kiểm tra và sửa lỗi.

> Nếu dữ liệu đạt chất lượng, pipeline tiếp tục tạo lớp Gold staging trên S3 và
> dùng Glue Load job để load dữ liệu vào Amazon Redshift. Trong Redshift, dữ
> liệu được tổ chức theo mô hình star schema gồm fact table cho chuyến đi và
> dimension table cho ngày, zone. Đây là mô hình phù hợp cho phân tích BI vì
> dashboard có thể query nhanh và dễ hiểu hơn.

> Cuối cùng, Metabase chạy local và kết nối tới Redshift để hiển thị dashboard.
> Người xem dashboard không cần biết toàn bộ logic phía sau, nhưng họ nhận được
> dữ liệu đã qua pipeline, đã kiểm tra chất lượng, và có cấu trúc phù hợp cho
> phân tích.

Close the diagram explanation with:

> Tóm lại, flow này thể hiện tư duy DataOps: dữ liệu đi qua nhiều lớp rõ ràng,
> mỗi bước có trách nhiệm riêng, có kiểm tra chất lượng trước khi publish, có
> quarantine khi lỗi, và có vòng lặp cải tiến liên tục khi team phát hiện vấn
> đề trong dữ liệu hoặc logic xử lý.

### Speaker Notes: How Data Is Processed In This Example

Use this as a more detailed narrative after the diagram.

Say:

> Trong ví dụ này, dữ liệu bắt đầu từ các file taxi trip. Mỗi record đại diện
> cho một chuyến xe, bao gồm thời gian pickup, thời gian dropoff, pickup zone,
> dropoff zone, khoảng cách, thời lượng và một số thuộc tính vận hành khác.

> Khi file được upload vào S3 Bronze, pipeline chưa giả định dữ liệu đã sạch.
> Bronze chỉ đóng vai trò là nơi lưu dữ liệu gốc. Việc giữ raw data giúp chúng
> ta có thể replay pipeline, backfill dữ liệu, hoặc so sánh lại kết quả nếu
> logic xử lý thay đổi.

> Ở bước xử lý Bronze sang Silver, Glue Spark đọc dữ liệu thô và chuẩn hóa nó.
> Pipeline có thể loại bỏ record không hợp lệ, chuẩn hóa kiểu dữ liệu ngày giờ,
> tính duration của chuyến đi, join với taxi zone lookup để bổ sung thông tin
> borough hoặc zone name, và ghi dữ liệu sạch hơn về S3 Silver.

> Sau đó, Data Quality đóng vai trò như một cổng kiểm soát. Đây là nơi chúng ta
> chuyển từ câu hỏi "job có chạy xong không?" sang câu hỏi quan trọng hơn:
> "dữ liệu tạo ra có đủ đúng để dùng không?". Nếu rule fail, pipeline dừng lại
> và đưa dữ liệu vào Quarantine. Nếu rule pass, dữ liệu mới được phép đi tiếp.

> Khi dữ liệu đã pass quality gate, Glue chuẩn bị dữ liệu Gold và load vào
> Redshift. Lúc này dữ liệu không còn chỉ là file nữa; nó trở thành data model
> phục vụ phân tích. Fact table lưu các chuyến đi, còn dimension table giúp
> người dùng phân tích theo ngày, tháng, pickup zone, dropoff zone hoặc borough.

> Metabase là lớp trình bày cuối cùng. Dashboard có thể trả lời các câu hỏi như:
> tổng số chuyến là bao nhiêu, nhu cầu thay đổi theo ngày hoặc theo giờ như thế
> nào, khu vực nào có nhiều pickup/dropoff nhất, route nào phổ biến, và dữ liệu
> đã được load đủ chưa.

> Vì vậy, điểm đáng chú ý không chỉ là chúng ta có dashboard. Điểm quan trọng là
> dashboard được xây trên một pipeline có kiểm soát: có raw layer, processed
> layer, quality gate, warehouse layer, và vòng lặp sửa lỗi khi phát hiện vấn
> đề.

Useful dashboard links when Metabase is running locally:

- Executive Overview: `http://localhost:3001/dashboard/4`
- Zone & Route Analysis: `http://localhost:3001/dashboard/5`
- Pipeline Proof: `http://localhost:3001/dashboard/6`

### Current Validated Result

The latest validated successful run loaded:

| Table | Rows |
| --- | ---: |
| `nyc_taxi.fact_fhvhv_trips` | 2,464,997 |
| `nyc_taxi.dim_date` | 60 |
| `nyc_taxi.dim_zone` | 265 |

Glue Data Quality result:

- Score: `1.0`
- Rules passed: `11/11`

### Speaker Notes: Current Validated Result

Say:

> Đây là phần chứng minh pipeline đã chạy thật và tạo ra dữ liệu thật. Fact
> table có hơn 2.4 triệu dòng chuyến đi, dim_date có 60 ngày, và dim_zone có
> 265 zone.

> Glue Data Quality đạt score 1.0 với 11 trên 11 rule pass. Điều này không có
> nghĩa dữ liệu hoàn hảo tuyệt đối, nhưng nó chứng minh dữ liệu đã vượt qua bộ
> kiểm tra chất lượng mà team định nghĩa trước khi đưa vào Redshift.

> Khi trình bày, nên dùng kết quả này để nhấn mạnh: DataOps không dừng ở kiến
> trúc trên giấy. Pipeline phải có kết quả validate được bằng số liệu, bảng dữ
> liệu và dashboard.

### AWS Services Used

| Service | Role in the architecture |
| --- | --- |
| Amazon S3 | Bronze, Silver, Gold staging, scripts, logs |
| AWS Glue Catalog | Metadata catalog for Bronze and Silver tables |
| AWS Glue Crawlers | Schema discovery |
| AWS Glue ETL | Spark transformations and Redshift load preparation |
| AWS Glue Data Quality | Silver data contract validation |
| AWS Step Functions | Pipeline orchestration |
| Amazon EventBridge | Optional scheduled trigger |
| Amazon Redshift | Gold warehouse for analytics |
| CloudWatch Logs | Runtime logs |
| Metabase | Local BI dashboard layer |

### Speaker Notes: AWS Services Used

Say:

> Bảng này giúp audience hiểu vì sao từng AWS service xuất hiện trong kiến trúc.
> Amazon S3 là data lake. Glue Catalog và crawler quản lý metadata. Glue ETL xử
> lý dữ liệu bằng Spark. Glue Data Quality kiểm tra contract của dữ liệu.

> Step Functions là bộ điều phối để pipeline chạy theo đúng thứ tự và có branch
> khi quality fail. Redshift là warehouse cho dữ liệu Gold. Metabase là BI layer
> chạy local để demo dashboard.

> Một điểm quan trọng là chúng ta dùng managed services để giảm phần vận hành
> thủ công. Thay vì tự quản lý cluster Spark hoặc scheduler riêng, AWS cung cấp
> các thành phần đó dưới dạng service.

### Implementation Walkthrough

#### 1. Provision Infrastructure

```bash
export AWS_PROFILE=cloud-user
cd nash-dataops-terraform/terraform
./bootstrap_remote_state.sh
terraform init
terraform apply
```

Expected outputs include:

- `data_bucket_name`
- `step_functions_state_machine_arn`
- `redshift_host`
- `redshift_database`
- `redshift_schema`
- `redshift_fact_table`

#### 2. Upload Manual Inputs

```bash
aws s3 cp ../data/fhvhv_trips/2024/01/fhvhv_tripdata.parquet \
  s3://$(terraform output -raw data_bucket_name)/bronze/fhvhv_trips/2024/01/fhvhv_tripdata.parquet

aws s3 cp ../data/fhvhv_trips/2024/02/fhvhv_tripdata.parquet \
  s3://$(terraform output -raw data_bucket_name)/bronze/fhvhv_trips/2024/02/fhvhv_tripdata.parquet

aws s3 cp ../data/taxi_zone_lookup.csv \
  s3://$(terraform output -raw data_bucket_name)/bronze/reference/taxi_zone_lookup.csv
```

#### 3. Start Pipeline

```bash
aws stepfunctions start-execution \
  --state-machine-arn "$(terraform output -raw step_functions_state_machine_arn)" \
  --name "manual-$(date +%Y%m%d%H%M%S)" \
  --input '{}'
```

#### 4. Verify Redshift

Use Redshift Data API or Metabase to verify:

```sql
select count(*) as fact_rows
from nyc_taxi.fact_fhvhv_trips;
```

#### 5. Start Metabase

```bash
cd ../metabase
cp .env.example .env
# Edit .env and set REDSHIFT_PASSWORD.
docker compose up -d
./setup-redshift-database.sh
./import-demo-dashboards.py
```

### Speaker Notes: Implementation Walkthrough

Say:

> Walkthrough này là runbook ngắn để triển khai và chạy demo. Đầu tiên, chúng ta
> dùng Terraform để tạo infrastructure. Sau đó upload dữ liệu đầu vào thủ công
> vào S3 Bronze. Tiếp theo, start Step Functions execution để pipeline chạy toàn
> bộ các bước.

> Sau khi pipeline chạy xong, chúng ta kiểm tra Redshift để xác nhận dữ liệu đã
> được load vào fact table và dimension table. Cuối cùng, Metabase được chạy
> local, kết nối tới Redshift và import dashboard demo.

> Khi nói phần này, không cần đọc từng command. Hãy giải thích logic vận hành:
> provision infrastructure, upload input, run pipeline, verify warehouse, open
> dashboard.

### Demo Dashboard Story

Use the dashboards in this order:

1. `Nash DataOps - Executive Overview`
   - Show total loaded volume.
   - Show daily and hourly demand patterns.
   - Show pickup/dropoff completeness.

2. `Nash DataOps - Zone & Route Analysis`
   - Show top pickup/dropoff zones.
   - Show borough flows.
   - Show top routes and longest common routes.

3. `Nash DataOps - Pipeline Proof`
   - Show date range.
   - Show source lineage.
   - Show duration quality band.
   - Show location completeness profile.

### Speaker Notes: Demo Dashboard Story

Say:

> Khi demo dashboard, nên đi theo một câu chuyện thay vì mở từng chart rời rạc.
> Dashboard đầu tiên là Executive Overview để trả lời câu hỏi lớn: pipeline đã
> load bao nhiêu dữ liệu, nhu cầu thay đổi theo ngày và giờ như thế nào, và dữ
> liệu pickup/dropoff có đủ không.

> Dashboard thứ hai là Zone & Route Analysis. Ở đây chúng ta đi sâu hơn vào
> không gian địa lý: zone nào có nhiều pickup, zone nào có nhiều dropoff, route
> nào phổ biến, và flow giữa các borough trông như thế nào.

> Dashboard thứ ba là Pipeline Proof. Mục tiêu không chỉ là phân tích business,
> mà là chứng minh pipeline đáng tin cậy: dữ liệu có date range rõ ràng, có
> lineage, có profile về duration và location completeness.

> Câu chuyện nên kết thúc bằng thông điệp: dashboard đẹp là phần nhìn thấy bên
> ngoài, nhưng giá trị thật nằm ở pipeline có kiểm soát phía sau.

### Final Talking Points

Use these points to close the presentation:

- The pipeline is automated end to end after manual input upload.
- Data quality is part of the flow, not an afterthought.
- Redshift stores a clean Gold model for analytics.
- Metabase dashboards are generated from code, making the BI layer repeatable.
- Terraform keeps infrastructure reproducible.
- Step Functions, Glue, Redshift, and Metabase together form a practical
  DataOps demo on AWS.

### Speaker Notes: Final Talking Points

Say:

> Để kết thúc, tôi muốn nhấn mạnh ba điểm. Thứ nhất, pipeline này tự động hóa
> luồng xử lý sau khi dữ liệu được upload thủ công. Thứ hai, data quality là một
> phần của pipeline, không phải bước kiểm tra thủ công sau cùng. Thứ ba, dữ liệu
> được đưa vào Redshift theo mô hình phù hợp cho analytics và được trình bày qua
> Metabase dashboard.

> Demo này cũng cho thấy DataOps là sự kết hợp giữa engineering, infrastructure,
> quality control và analytics. Terraform giúp infrastructure có thể tái tạo,
> Step Functions giúp orchestration rõ ràng, Glue xử lý và kiểm tra dữ liệu,
> Redshift lưu Gold model, còn Metabase giúp business user nhìn thấy giá trị.

> Nếu chỉ nhớ một câu, hãy nhớ rằng DataOps không chỉ là chạy pipeline. DataOps
> là cách tổ chức pipeline để dữ liệu có thể tin được, giải thích được và cải
> tiến được.

## Appendix: Useful Commands

Check Step Functions execution:

```bash
aws stepfunctions describe-execution \
  --execution-arn <execution-arn>
```

Check latest Glue job run:

```bash
aws glue get-job-runs \
  --job-name glue-process-bronze-to-silver-dev \
  --max-results 1
```

Check Terraform drift:

```bash
cd nash-dataops-terraform/terraform
terraform plan
```

Re-import dashboards:

```bash
cd nash-dataops-terraform/metabase
./import-demo-dashboards.py
```

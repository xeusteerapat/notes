แนวทางประหยัด GitHub Copilot AI Credits สำหรับ Existing Codebase

ถ้างานส่วนใหญ่ของเราเป็นการ ต่อยอด feature ที่มีอยู่แล้ว ใน codebase เดิม สิ่งที่กิน AI Credits เยอะจริง ๆ มักไม่ใช่การให้ model “เขียน code”

แต่คือการที่ model ต้อง ทำความเข้าใจ architecture, abstraction และ convention เดิมซ้ำ ๆ

โดยเฉพาะ codebase ที่มี pattern ประมาณนี้:

* React component factory
* Dynamic form composition
* Higher-order function
* Curried validator
* Higher-order component
* Abstraction ซ้อนหลายชั้น
* Internal convention ที่ไม่เหมือน React ทั่วไป

ใน codebase แบบนี้ แนวคิดที่ช่วยประหยัด credit ได้มากที่สุดคือ:

ยอมจ่ายครั้งหนึ่งเพื่อสอน model ให้เข้าใจ codebase แล้ว reuse ความรู้นั้นในงานถัด ๆ ไป

⸻

1. อย่าให้ Model ต้อง Rediscover Architecture ซ้ำทุกครั้ง

สมมติว่า form component ใน project ถูกสร้างด้วย flow ประมาณนี้:

createField(config)
  -> withValidation(validators)
  -> withTracking()
  -> ReactComponent

ถ้า AI agent ไม่มี context มาก่อน มันอาจต้องทำสิ่งเหล่านี้ใหม่ทุกครั้ง:

1. หา existing field
2. อ่าน implementation ของ factory
3. ไล่ chain ของ HOF
4. ทำความเข้าใจ validator composition
5. เปิดดู test
6. เดา convention ของ project
7. แล้วค่อย implement feature

ถ้าทุก task ต้องผ่านขั้นตอนนี้ใหม่ทั้งหมด token จำนวนมากจะถูกใช้ไปกับการ ทำความเข้าใจ architecture เดิมซ้ำ ๆ

สิ่งที่ควรทำคือ document architectural invariant สำคัญเอาไว้ครั้งเดียว

⸻

2. ใช้ Repository Instructions

สร้างไฟล์:

.github/copilot-instructions.md

ไฟล์นี้เอาไว้บอก knowledge ระดับ project ที่ Copilot ควรรู้

ตัวอย่าง:

# Project Architecture
This project uses factory-based React components.
Prefer extending existing patterns instead of introducing new abstractions.
General principles:
- Find the closest existing implementation before writing new code.
- Prefer minimal changes.
- Do not refactor unrelated modules.
- Follow existing factory/HOF patterns even when a simpler React implementation exists.
- Use the nearest existing tests as the reference for new tests.

เป้าหมายไม่ใช่การเขียน documentation ให้ครบทุกอย่าง

แต่คือการเก็บข้อมูลที่ช่วยให้ agent ไม่ต้อง rediscover การตัดสินใจทาง architecture เดิมซ้ำ

⸻

3. ใช้ Path-Specific Instructions

บาง knowledge ใช้เฉพาะบาง directory เท่านั้น

แทนที่จะยัดทุกอย่างไว้ใน global instruction ให้แยกตามพื้นที่ของ codebase

ตัวอย่าง structure:

.github/
├── copilot-instructions.md
│
├── instructions/
│   ├── forms.instructions.md
│   ├── validators.instructions.md
│   └── tests.instructions.md
│
└── prompts/
    ├── extend-feature.prompt.md
    ├── debug-memory.prompt.md
    └── debug-k8s.prompt.md

ตัวอย่าง:

---
applyTo: "src/forms/**/*.{ts,tsx}"
---
Forms in this project use factory-based composition.
Typical flow:
createField(config)
  -> withValidation(validators)
  -> withTracking()
  -> React component
When implementing a new field:
1. Find the closest existing field.
2. Reuse its factory chain.
3. Change only field-specific configuration and validation.
4. Do not convert factory components into standard React components.
5. Do not introduce new abstractions unless required.
6. Do not refactor unrelated code.
7. Add tests following the nearest existing field test.

ข้อดีคือ agent จะได้รับ context เฉพาะตอนที่ทำงานกับ code ส่วนนั้น

ไม่ต้องแบก knowledge ทุกเรื่องของทั้ง repo ไปทุก prompt

⸻

4. อ้างอิง Existing Implementation ให้ชัด

สำหรับงาน extend feature อย่าใช้ prompt กว้าง ๆ แบบ:

Implement a new Customer ID field.

เพราะมันเปิด search space กว้างมาก

agent อาจต้องไปสำรวจเองว่า:

* form ถูกสร้างยังไง
* validator อยู่ไหน
* pattern ที่ถูกต้องคืออะไร
* test เขียนแบบไหน

ควรเขียนประมาณนี้แทน:

Implement CustomerIdInput following the same pattern as CustomerNameInput.
Reuse the existing field factory and validator chain.
Only change field-specific configuration and validation.
Do not introduce new abstractions.

existing implementation จะกลายเป็น shortcut ให้ model

แทนที่จะถามว่า:

Project นี้ควร implement feature นี้ยังไง?

เรากำลังบอกว่า:

ใช้ pattern ที่มีอยู่แล้ว แล้วเปลี่ยนเฉพาะส่วนที่จำเป็น

ซึ่งโดยทั่วไปทั้งประหยัด token และลดโอกาส hallucinate architecture ใหม่ขึ้นมาเอง

⸻

5. สร้าง Reusable Prompt Files

งานบางประเภทเราทำซ้ำบ่อยมาก เช่น:

* Extend existing feature
* Debug memory
* Debug Kubernetes/OpenShift
* Generate tests
* Review implementation

สามารถสร้าง prompt reuse ได้

ตัวอย่าง:

.github/prompts/extend-feature.prompt.md
Implement the requested feature by extending an existing pattern.
Before editing:
1. Find the closest existing implementation.
2. Identify the factory/HOF chain it uses.
3. Reuse that pattern.
Constraints:
- Prefer minimal changes.
- Do not introduce new abstractions.
- Do not refactor unrelated code.
- Do not scan unrelated directories.
- Run only relevant tests.
If multiple implementations appear plausible, compare them briefly before making changes.

ข้อดีไม่ใช่แค่ไม่ต้องพิมพ์ prompt ซ้ำ

แต่ prompt แบบนี้บังคับให้ agent ใช้ bounded search strategy

แทนที่จะสำรวจ repo แบบกว้าง ๆ

⸻

6. ใช้ Model ราคาถูกกับงานที่เป็น Pattern-Following

สำหรับงาน routine ที่มี reference implementation ชัดอยู่แล้ว ควรเริ่มจาก model ราคาถูกก่อน

workflow ที่ใช้งานได้จริงประมาณนี้:

GPT-5.6 Luna
     ↓
Claude Sonnet 5
     ↓
GPT-5.6 Sol

GPT-5.6 Luna

เหมาะกับงานประมาณนี้:

* เพิ่ม field ใหม่ตาม existing field
* เพิ่ม validator
* แก้ TypeScript type
* เขียน unit test แบบตรงไปตรงมา
* เพิ่ม endpoint เล็ก ๆ
* Boilerplate
* Simple refactor
* Explain local code

ถ้า architecture ถูก document ไว้ดีแล้ว งานจำนวนมากจะกลายเป็น pattern matching

ตรงนี้คือจุดที่ model ราคาถูกจะคุ้มมาก

⸻

Claude Sonnet 5

ขยับมาใช้เมื่อ task ต้อง reasoning กับ repo มากขึ้น เช่น:

* abstraction หลายชั้น
* หลาย module เกี่ยวข้องกัน
* factory chain ไม่ตรงไปตรงมา
* feature ใหญ่
* refactor ที่มีผลหลายจุด
* debug ข้ามหลาย file
* agent ต้อง explore repository พอสมควร

⸻

GPT-5.6 Sol

เก็บไว้ใช้กับปัญหาที่ reasoning ยากจริง ๆ เช่น:

* memory leak
* race condition
* production incident
* architecture migration
* bug ที่ลองแก้มาหลายวิธีแล้วยังไม่ได้
* root-cause analysis ข้ามหลายระบบ

แนวคิดไม่ใช่:

cheap model = easy question
expensive model = difficult question

แต่ควรคิดเป็น:

known pattern
    → cheap model
unknown architecture
    → stronger model
hard root-cause reasoning
    → strongest model

⸻

7. สอน Architecture ครั้งเดียว

บางครั้งการยอมใช้ model แพงกว่าครั้งหนึ่งถือว่าคุ้ม

เช่นใช้ model ที่ reasoning ดีกว่าให้ช่วย reverse-engineer module ที่ซับซ้อน

ตัวอย่าง prompt:

Analyze the form architecture in this repository.
Focus on:
- component factories
- validator composition
- higher-order functions
- component lifecycle
- state flow
- testing conventions
Do not change any code.
Produce a concise list of architectural invariants that another coding agent would need in order to extend this module safely.

สมมติ model สรุปได้ว่า:

Every input component is created through createField.
Validation functions are curried.
Validators execute left-to-right and stop on the first error.
Tracking is attached after validation.
All fields expose the same FieldProps interface.
Field tests mock the validator factory rather than the final validator.

เอาสิ่งเหล่านี้ไปเก็บไว้ใน:

.github/instructions/forms.instructions.md

หลังจากนั้น agent ตัวอื่นไม่ต้อง reverse-engineer ใหม่

แนวคิดนี้มองได้ว่าเป็น:

Architecture Caching

⸻

8. งาน Debug ควรใช้ Workflow คนละแบบ

Feature development กับ debugging มีพฤติกรรมการกิน token ไม่เหมือนกัน

เวลา debug สิ่งที่กิน token มากที่สุดมักเป็น raw telemetry

เช่น:

20,000 lines of logs

หรือ:

kubectl describe everything

หรือ:

Grafana export ทั้งก้อน

แทนที่จะ dump ทุกอย่างให้ model ควรลด search space ก่อน

⸻

9. ใช้ Hypothesis-Driven Debugging

แทนที่จะส่งแบบนี้:

My Node.js app has a memory leak.
Here are 30,000 lines of logs.

ควรส่ง observation ที่สรุปแล้ว:

Node.js 20
Axios requests with:
- keepAlive=true
- concurrency=1000
- timeout=0
Observed:
RSS continuously increases.
heapUsed remains mostly stable.
external memory increases.
completed requests remain near zero when upstream requests hang.
Investigate whether the growth could be caused by sockets,
pending requests, or external buffers rather than V8 heap objects.

แบบนี้ model จะมี hypothesis space แคบลงมาก

แทนที่จะคิดถึงทุกสาเหตุของ Node.js memory issue

มันสามารถ focus ที่:

RSS
 ├── V8 heap
 ├── native allocations
 ├── Buffer / external memory
 ├── sockets
 └── runtime overhead

ได้ทันที

⸻

10. Filter Kubernetes / OpenShift Evidence ก่อน

Production incident เป็นอีก use case ที่ context บวมเร็วมาก

แทนที่จะส่ง log ทั้ง pod ควร filter ก่อน

เช่น:

oc logs <pod> --since=15m
oc logs <pod> --previous
oc describe pod <pod>
oc get events --sort-by=.lastTimestamp

จากนั้นสรุปเฉพาะช่วงเวลาที่เกี่ยวข้อง

ตัวอย่าง:

Timeline:
10:31 request spike starts
10:32 memory jumps from 500 MB to 1.5 GB
10:33 readiness probe fails
10:34 pod receives OOMKilled
10:35 pod restarts
Relevant logs:
...
Resource limits:
requests:
  memory: 512Mi
limits:
  memory: 2Gi

timeline แบบนี้มักมีประโยชน์กว่า raw logs หลายพันบรรทัด

⸻

11. หนึ่ง Conversation ต่อหนึ่ง Feature หรือ Incident

สำหรับ memory issue ก็ควรให้ thread นั้น focus เรื่อง memory

เช่น:

memory-leak-investigation

อย่าเอาไปผสมกับ:

React feature
MongoDB optimization
CI configuration

conversation ที่ยาวเกินไปจะสะสม context เก่า

แต่ในทางกลับกัน ถ้าเปิด conversation ใหม่ทุก 5 นาที model ก็ต้อง rebuild context ใหม่ตลอด

rule ที่ใช้ง่ายคือ:

One conversation per feature or incident.

พอ task จบ ค่อยเอาความรู้ที่ durable ไปเก็บใน instructions หรือ documentation

⸻

12. ให้ Evidence แทนที่จะให้ Agent Explore เอง

เวลาถามเรื่อง debug ควรให้ observed facts ก่อน

เช่น:

Observed:
- RSS grows
- heapUsed stable
- active sockets increase
- upstream hangs

ถูกกว่าการบอกว่า:

Inspect the entire repository and determine why memory is increasing.

โครงสร้าง debugging prompt ที่ดีควรมี:

Environment
Symptoms
Timeline
Metrics
Recent changes
Relevant code
Current hypothesis
Question

ตัวอย่าง:

Environment:
Node.js 20
Axios 1.x
OpenShift
Symptoms:
RSS grows from 300 MB to 1.8 GB.
heapUsed remains around 250 MB.
External memory grows steadily.
Recent change:
Added persistent HTTP agents using keepAlive=true.
Traffic:
~1000 concurrent requests.
Upstream behavior:
Some requests never respond.
Question:
Which memory categories or resources could explain RSS growth
without corresponding heapUsed growth?

แบบนี้พอเราต้อง escalate ไป model แพงกว่า มันก็ใช้ reasoning กับข้อมูลที่มีคุณภาพแทนที่จะเสีย token ไปกับการค้นหาเอง

⸻

13. จำกัด Repository Exploration

Agent mode สามารถใช้ token จำนวนมากกับการ search code

แทนที่จะบอกว่า:

Find how validation works in this project.

ควรบอกว่า:

Validation is implemented primarily under:
src/forms
src/validators
Start with:
src/forms/customer/
src/validators/createValidator.ts
Do not search unrelated directories unless these files reference them.

มอง repository เหมือน search space

ยิ่ง search space เล็ก agent ก็ยิ่งใช้ context น้อย

⸻

14. Run เฉพาะ Test ที่เกี่ยวข้องก่อน

ไม่ควรให้ agent run test suite ทั้งก้อนทุกครั้งที่แก้ code เล็กน้อย

ใช้แบบนี้ก่อน:

Run the nearest unit test first.
Only run the broader test suite if the focused test passes.

เช่น:

npm test -- CustomerNameInput

ก่อนที่จะรัน:

npm test

ช่วยลดทั้ง:

* tool calls
* execution time
* logs
* context ที่ model ต้องอ่าน

⸻

15. ห้าม AI Refactor เกิน Scope

AI agent มีนิสัยอย่างหนึ่งคือเห็น architecture แปลกแล้วอยาก “แก้ให้ดีขึ้น” 😆

แต่ใน legacy system นี่มักเป็นสิ่งที่เราไม่ต้องการ

ใส่ constraint ไปตรง ๆ ว่า:

Do not improve the architecture.
Follow the existing pattern even if another implementation
would normally be simpler or more idiomatic.

ใน existing system หลายครั้ง:

Consistency สำคัญกว่า Elegance

โดยเฉพาะตอน extend feature ที่มีอยู่แล้ว

⸻

Recommended Workflow สำหรับ Feature Development

Requirement
   ↓
หา existing feature ที่ใกล้ที่สุด
   ↓
ชี้ reference implementation ให้ agent
   ↓
Load path-specific instructions
   ↓
GPT-5.6 Luna
   ↓
Focused tests
   ↓
Done

ถ้า Luna เริ่มไม่ไหว:

Luna
  ↓
Sonnet 5

ถ้ากลายเป็น architecture หรือ root-cause problem:

Sonnet 5
   ↓
GPT-5.6 Sol

⸻

Recommended Workflow สำหรับ Debugging

Raw telemetry
      ↓
Filter evidence
      ↓
สร้าง timeline
      ↓
สรุป observations
      ↓
สร้าง hypotheses
      ↓
ถาม focused question
      ↓
Escalate model ถ้าจำเป็น

จาก:

20,000 log lines

พยายามเปลี่ยนให้กลายเป็น:

50 relevant lines
      ↓
5 observations
      ↓
3 hypotheses
      ↓
1 focused investigation

การลดข้อมูลแบบนี้มักประหยัด token ได้มากกว่าการพยายามย่อ prompt ทีละไม่กี่บรรทัดเสียอีก

⸻

หลักสำคัญที่สุด

การ optimize AI Credits ที่ดีที่สุดไม่ใช่:

เขียน prompt ให้สั้นที่สุด

แต่คือ:

ลดปริมาณสิ่งที่ model ต้องค้นหาและทำความเข้าใจใหม่

สำหรับ codebase ที่มี factory, HOF, composition pattern หรือ legacy abstraction:

Discover once
     ↓
Document invariants
     ↓
Reuse instructions
     ↓
Reference existing implementations
     ↓
Use cheaper models
     ↓
Escalate เฉพาะตอนที่ reasoning ยากจริง

สรุปเป็นประโยคเดียว:

ยอมจ่ายครั้งหนึ่งเพื่อสอน model ให้เข้าใจ codebase แล้ว reuse ความรู้นั้นให้ได้มากที่สุด

# Common Interview Mistakes

This guide expands on common mistakes made by candidates and interviewers during mobile system design interviews, specifically focusing on iOS and Android platforms. The aim is to provide candidates with a practical understanding of expectations and help them prepare effectively.

## As a Candidate

### Not Being Prepared

Preparation is key. Don't just memorize patterns; understand the *why* behind them.

- **Study Open-Source Projects:**
    - **Beyond the Code:** Look beyond the UI and try to understand the architectural choices, networking layers, data persistence strategies, and error handling.
    - **Platform-Specific Patterns:** Pay attention to iOS-specific patterns like `Coordinator` pattern, `Combine` framework, or `SwiftUI` integration with `UIKit`. For Android, focus on `MVVM` with `LiveData`/`Flow`, `Coroutines`, `Jetpack Compose`, and dependency injection frameworks like `Hilt` or `Dagger`.
    - **Example Resources:**
        - **iOS:** Look into popular open-source iOS apps or libraries on GitHub (e.g., a well-architected media player or a networking library). Analyze how they handle background tasks, push notifications, or data synchronization.
        - **Android:** Explore apps like AntennaPod or projects using the Architecture Components. Focus on understanding background processing (WorkManager), UI rendering (RecyclerView optimizations), and data management (Room, Retrofit).
    - **Focus on Architecture:** Identify common architectures like `MVVM`, `MVP`, `Clean Architecture`, and understand their trade-offs in the context of mobile development.

- **Study Company Dev Blog:**
    - **Tech Stack Insights:** Company blogs often reveal details about their tech stack, chosen architectural patterns, and solutions to specific technical challenges.
    - **Identify Priorities:** Understand the company's priorities, such as performance optimization, security, or scalability, which might be reflected in their interview questions.
    - **Specific Technologies:** Look for articles about the specific languages (Swift/Kotlin), frameworks (`UIKit`/`SwiftUI` vs. `Compose`), and libraries they use. If they've written about challenges with certain technologies, that's a *major* hint.
    - **Example:** If a company's blog discusses their migration from Objective-C to Swift, be prepared to discuss Swift interoperability, memory management, and the benefits of Swift.

- **Conduct Mock Interviews with your peers:**
    - **Realistic Scenarios:** Simulate real-world scenarios with your friends. This could include designing a chat application, a news feed, or an e-commerce platform. Ensure you cover areas like offline support, data synchronization, and push notifications.
    - **Platform-Specific Concerns:** In iOS mock interviews, consider questions around memory management (reference cycles), UI responsiveness, background execution limits, and Core Data/Realm integration. For Android, address issues like activity lifecycle management, background services, battery optimization, and handling different screen sizes and densities.
    - **Time Management:** System design questions can be broad.  Practice timeboxing each part of the discussion (requirements gathering, high-level design, component breakdown, etc.).
    - **Communication is Key:** The mock interviewer should provide feedback not only on your technical solution but also on your communication skills, clarity, and ability to explain complex concepts.

- **Practice Handling Novelty:**
    - **Don't Freeze on Niche Topics:** If asked to design something unexpected (e.g., an SDK, a Library, or Server-Driven UI), don't panic. Fall back to first principles: Define the API, the Data Format, and the Storage.
    - **Practice by Doing:** Do not just read guides. Open the App Store, pick a random feature from a top 10 app (e.g., Spotify Player, Uber Map), and spend 30 minutes trying to design it yourself. This builds the intuition needed to handle unique problems rather than reciting memorized solutions.

### Rushing Towards a Solution

Take a step back and think!

- **Not Gathering System Requirements:**
    - **User Base:** "How many users will the app support initially? What is the expected growth?"
    - **Platform:** "Are we targeting only iOS or Android, or both?  What are the minimum supported OS versions?"
    - **Features:** "Which features are core to the app's functionality? Which are optional or can be deferred?"
    - **Non-Functional Requirements (NFRs):** "What are the key non-functional requirements like performance, security, scalability, and reliability?  Are there any specific privacy concerns (GDPR, HIPAA)?"
    - **Data Volume & Constraints:** Don't just ask about users; ask about data. "What is the maximum file size for uploads?" (Critical for apps like Dropbox). "Are we storing stock history for 1 day or 10 years?"
    - **Example:** Don't assume "build a chat app."  Ask about:  Message volume?  Media support? Group chats? End-to-end encryption? Offline access?

- **Not Asking Clarifying Questions:**
    - **Target Market Constraints:** Understand the geographical location. If the target is a developing market, you must design for spotty internet, high data costs, and lower-end device storage. Consider peer-to-peer sharing (Bluetooth) or data-saver modes.
    - **Audience Size:** "Is this for a small team, a large enterprise, or the general public?" The scale of the audience will impact the architectural decisions around backend infrastructure and scalability.
    - **Dev Team Details:** "What is the size and expertise of the development team?" (A large team requires more modularization to avoid merge conflicts; a small team benefits from simplicity).
    - **Device Capabilities:** "What is the minimum device capability we want to support? This will determine the memory available, processing power and supported OS features. It is important for memory management and performance optimizations."
    - **Product Specifics:** If designing a known product (e.g., Trello, Spotify), do not assume you know how it works. Ask: "Do columns move?" "Does the music stop if I go offline?"
    - **Example:** "What kind of data are we storing? Is it sensitive personal data that requires encryption at rest?"

- **Jumping Straight to Implementation:**
    - **High-Level Design First:** Start with a high-level overview of the system architecture, outlining the key components, their responsibilities, and how they interact with each other.
    - **Avoid the "UI Trap":** A common mistake is spending 20 minutes drawing ViewControllers, Screens, or Lists. This provides very little system design signal. Focus on the "Hard Parts" (Sync Engine, Pagination Logic, Conflict Resolution) instead of the UI layer.
    - **UI Architecture Patterns:** Delay discussing specific view classes or UI architecture patterns (MVC, MVVM, VIPER, MVI) until you've established the overall system design.
    - **Example:** Instead of immediately discussing `UITableView` or `RecyclerView`, talk about the overall data flow, data models, networking layer, and caching strategy.
    - **Vendor-Specific Solutions:** Avoid mentioning vendor-specific solutions at this stage. For example, don't say "We'll use Firebase for push notifications" before discussing the overall push notification architecture. Discuss the requirements and different potential approaches for achieving the goal.

### The "GitHub Template" Syndrome

One of the most frequent negative signals is trying to fit a pre-memorized solution into a specific problem.

- **Avoid Generic Diagrams:** Drawing standard boxes labeled "RemoteDataSource," "Repository," "UseCase," and "ViewModel" provides zero signal because almost every app has these. It looks like you memorized a template from a guide.
- **System Components > Code Patterns:** Instead of class diagrams, draw *System Components* that match the specific problem. For example: "Sync Engine," "Upload Manager," "Conflict Resolver," or "Image Processing Queue."
- **Justify Abstractions:** Do not add a "Repository" or "Dependency Injection" box just because a blog post said so. If the app is a simple map viewer, a complex repository layer might be over-engineering.

### Scope Creep & Over-Engineering

Balancing what to build vs. what *not* to build is a senior skill.

- **Scope Creep:** Do not voluntarily add complex features that weren't asked for (e.g., "Let's also add video editing," "Let's add AI recommendations," or "Let's add real-time user status"). This invites hard questions you might not be ready for and wastes valuable time.
- **Failure to Negotiate:** If the stakeholder (interviewer) asks for a complex feature with low business value (e.g., "Real-time updates for every 'like' on a post"), a Senior/Staff engineer should push back. Explain the high technical cost vs. low user value and propose a simpler alternative (eventual consistency).
- **YAGNI (You Ain't Gonna Need It):** Don't design for hypothetical futures (e.g., "What if we need to support VR headsets later?"). Focus on the MVP first.
- **Avoid "Rube Goldberg" Solutions:** Don't create complex chains of events (e.g., "Task/Job/Query/Command" architectures) for simple problems. Start simple and add complexity only when the requirements demand it.

### Being Unresponsive

Communicate clearly and constantly.

- **Keeping Silence:**
    - **Think Aloud:** Verbally articulate your thought process, even if you're unsure of the solution. Silence provides no signal to the interviewer.
    - **Explain Your Reasoning:** Explain the rationale behind your design decisions, highlighting the trade-offs and alternatives considered.
    - **Example:** Instead of silently pondering, say, "Okay, I'm thinking about using a message queue for handling background tasks. This would allow us to decouple the UI from the task execution and improve responsiveness. However, it also adds complexity. Let me explain further."

- **Waiting Until the Interviewer Starts Asking Questions:**
    - **Drive the Discussion:** Take initiative by presenting your solution in a structured manner, covering all key aspects of the design.
    - **Anticipate Questions:** Anticipate potential questions from the interviewer and proactively address them in your explanation.
    - **Example:** After presenting the high-level architecture, say, "Now, I'd like to dive into the data synchronization strategy and discuss how we can handle conflicts. This is an area where we need to consider different approaches based on the requirements."

### Giving Up Early

Don't let a single roadblock derail you.

- **Persistence is Key:** System design interviews are challenging. If you encounter a problem, don't give up. Try to identify alternative approaches or break down the problem into smaller, more manageable parts.
- **Never say "We can't do anything":** If asked a tough edge case (e.g., "What if the user never connects to the internet?"), do not say "Then we can't do anything." Speculate on alternatives: Desktop sync? Peer-to-peer sharing? SD card transfer?
- **Focus on the Process:** Even if you don't arrive at the perfect solution, demonstrate your problem-solving skills, critical thinking, and ability to learn from mistakes.

### Talking Too Much & "Title Dropping"

Be concise and relevant.

- **Long Introduction:**
    - **Focus on Relevance:** Keep your introduction brief (1-2 minutes). Focus on the experiences that are directly relevant to the role (e.g., mobile dev) rather than discussing irrelevant history (e.g., C++ gaming from 10 years ago).
    - **Highlight Key Achievements:** Highlight key achievements and projects that demonstrate your expertise in mobile system design, architecture, and development.

- **Avoid Self-Aggrandizement:** Avoid introducing yourself as a "Software Architect," "Expert," or "Guru". It invites the interviewer to grill you harder to prove the title. Be humble and describe your actual responsibilities.

- **Trying to Force an Interviewing Framework:**
    - **Adapt to the Interviewer:** Be flexible and adapt to the interviewer's style and preferences. Don't rigidly adhere to a specific framework if it doesn't align with the flow of the conversation.
    - **Use as a Guide:** Treat frameworks as a guide, not a strict protocol.

- **Ignoring the Interviewer:**
    - **Listen Carefully:** Pay close attention to the interviewer's questions, comments, and suggestions. They may be providing valuable hints or guidance.
    - **Pause for Interruption:** If the interviewer interrupts you, stop talking and listen to what they have to say. Don't try to finish your thought at the expense of their input.

- **Repeating Yourself:**
    - **Move Forward:** Once you've clearly explained a concept or decision, move on to the next topic. Don't reiterate the same points without adding new information.

- **Jumping from Topic to Topic:**
    - **Structured Approach:** Maintain a logical flow in your presentation, covering each topic in a clear and organized manner.
    - **Transition Smoothly:** Use transitional phrases to guide the interviewer through your thought process and indicate when you're moving on to a new topic.

- **Going Too Broad with the Answers:**
    - **Stay Focused:** Answer the questions directly and avoid tangents that are not relevant to the topic at hand.
    - **Provide Meaningful Details:** Provide sufficient detail to demonstrate your understanding of the concepts, but avoid getting bogged down in unnecessary minutiae.

### Treating the Interviewer as an Adversary

The interviewer is on your side (or should be!).

- **Collaboration, Not Confrontation:** Remember that the interviewer is trying to assess your skills and potential fit within the team. Treat the interview as a collaborative problem-solving exercise, not an adversarial interrogation.

### Trying to Fit a Solution into an Existing Scheme

Don't force a square peg into a round hole.

- **Tailor Your Approach:** Design your solution based on the specific requirements of the problem, not on pre-conceived notions or existing designs.
- **Flexibility is Key:** Be flexible and willing to adapt your approach if new information emerges or the interviewer provides feedback.

### Being Toxic

No one wants to work with someone unpleasant.

- **Being Opinionated:**
    - **Open-Mindedness:** Acknowledge that there are multiple valid approaches to solving a problem and be open to considering alternatives.
    - **Justify Your Choices:** Provide a rationale for your preferred approach, but avoid dismissing other options out of hand.

- **Interrupting the Interviewer:**
    - **Respectful Communication:** Allow the interviewer to finish their thought before interjecting with your own opinions or questions.

- **"Educating" the Interviewer:**
    - **Humility:** Avoid lecturing the interviewer or assuming that you know more than they do.
    - **Polite Disagreement:** If you disagree with something the interviewer says, express your disagreement politely and respectfully, providing evidence or reasoning to support your point of view.

- **Being a "Pleaser":**
    - **Authenticity:** Be genuine and authentic in your responses. Don't try to tell the interviewer what you think they want to hear.

### Technical Competence & Justification

Show your thought process and avoid "faking it."

- **Provide Justification:** Always explain the reasoning behind your design decisions.
    - **Avoid "Documentation" Answers:** Don't say "I used MVC because Apple recommends it." Say "I used MVC because it separates concerns, allowing us to test the logic independently of the UI."
    - **Avoid "Simplicity" as a Crutch:** Don't say "I used Shared Preferences because it's simple/native" if the requirement (e.g., large dataset) clearly demands a Database.

- **Faking Knowledge:**
    - **Don't Bluff:** If you don't know a specific technology (e.g., "How does Protobuf schema evolution work?"), **admit it**. Trying to bluff or dodge the question is a major red flag.
    - **Don't Guess:** If asked about backend databases, don't guess "Cassandra on mobile." Admit you aren't sure about the specific vendor but explain the *properties* you need (e.g., NoSQL, key-value store).

- **Buzzword Dropping:**
    - **Depth over Keywords:** Don't throw out terms like "GraphQL," "WebSockets," or "Clean Architecture" unless you can explain exactly *why* they solve a specific problem in *this* design better than the alternatives.
    - **Know Your Tools:** Using "Dependency Injection" for "performance" is a wrong answer. Know the real purpose of the patterns you cite.

- **Technical Missteps:**
    - **Client vs. Server:** Do not suggest performing heavy, trusted logic (e.g., conflict resolution, timestamp generation for ordering) on the client. The client is unreliable and clock-skewed.
    - **Security:** Do not suggest "Asymmetric Encryption" for local storage (it's slow/overkill). Do not suggest storing sensitive data if you can just keep it in RAM.
    - **Consider Alternatives:** For each key decision, briefly discuss alternative approaches and explain why you chose the specific solution you did.

### Lying

Honesty is always the best policy.

- **Be Truthful:** Never exaggerate your skills or experience. If you don't know the answer to a question, it's better to admit it than to make something up.
- **Focus on What You Know:** Focus on highlighting your strengths and areas of expertise, rather than trying to cover up your weaknesses.

### Lack of Decisiveness (Asking for Permission)

- **Be the Driver:** The most common behavioral mistake is constantly asking: "Is this okay?", "Should we support offline mode?", or "Do you think I need a database?"
- **State Your Intent:** A Senior/Staff candidate leads. Say: "I am going to support offline mode because this is a music player and users expect it," or "I will use UUIDs for IDs to prevent collisions."
- **Avoid "Hint Seeking":** Constantly looking to the interviewer for validation or hints signals a lack of confidence.

### Not Having Any Structure for the Solution

Organize your thoughts.

- **Prepare an Outline:** Before the interview, prepare a general outline of the topics you want to cover.
- **Time Management (Mental Timer):**
    - **Requirements:** ~5 minutes. (Don't spend 15 minutes here).
    - **High-Level Design:** ~10-15 minutes.
    - **Deep Dive:** ~20-25 minutes.
- **Use a Framework:** Consider using a system design framework as a guide, but be flexible and adapt it to the specific requirements of the problem.

### Speaking in Terms of Specific Vendors

Focus on core principles.

- **Abstract Away Implementation Details:** Focus on the underlying concepts and principles of system design, rather than getting bogged down in the specifics of particular vendors or technologies.
- **Vendor-Agnostic Language:** Use vendor-agnostic language to describe your solutions, focusing on the functionality and behavior of the system.

### Being Overly Optimistic

Consider edge cases and failure scenarios.

- **Think About Failure:** Consider potential failure scenarios, such as network outages, server errors, or device crashes, and design your system to handle these situations gracefully.
- **Error Handling:** Discuss error handling strategies, such as retries, fallbacks, and circuit breakers, to ensure the reliability and resilience of the system.
- **Example:** "What happens if the user loses network connectivity while uploading a large file? How do we handle data inconsistencies if there's a conflict between the local cache and the server?"
- **Backward Compatibility:** Specifically for SDKs or new features, ask: "What happens if the user has an old version of the app?" "How do we handle API changes without breaking existing clients?"

### Making Assumptions

Don't guess; ask.

- **Clarify Ambiguities:** If you're unsure about something, ask clarifying questions to ensure you understand the requirements and constraints of the problem.
- **State Your Assumptions:** If you have to make assumptions, explicitly state them and explain why you're making them.

### Ignoring the Interviewer's Hints

The interviewer wants you to succeed.

- **Pay Attention to Clues:** The interviewer may provide hints or suggestions to guide you in the right direction. Pay attention to these clues and use them to refine your solution.
- **Ask for Clarification:** If you're unsure about a hint, ask the interviewer to clarify it or provide more context.

## As an Interviewer

Your role is to evaluate fairly and help the candidate showcase their abilities.

### Not Being Prepared

- **Well-Defined Questions:** Have a clear understanding of the problem you want the candidate to solve and the key aspects of system design you want to evaluate.
- **Potential Discussion Topics:** Anticipate potential discussion topics and prepare follow-up questions to probe the candidate's understanding and explore different aspects of the design.
- **Ideal Solutions:** Have a general idea of the ideal solution and the trade-offs involved, but be open to alternative approaches and creative solutions.
- **iOS & Android Specifics:** For mobile system design, you *must* understand the OS specific constraints (Battery, Network, Lifecycle, Storage limits) and APIs to realistically assess a candidate. If you are a backend engineer interviewing a mobile candidate, ensure you don't penalize them for client-side specific patterns you aren't familiar with.

### Understanding Levels

- **Differentiate Candidates:**
    - **L4 (Mid-level):** Focus on the "Happy Path." Can they build a working solution?
    - **L5 (Senior):** Focus on the "Unhappy Path." Bad network, low disk space, edge cases, conflict resolution.
    - **L6 (Staff):** Focus on "Business & Crisis." Privacy lawsuits, team bottlenecks, OS breaking changes, long-term maintenance, and business value.
- **Allow "Stubbing":** If a candidate doesn't know a specific backend detail (e.g., specific DB sharding), allow them to abstract it ("Let's assume the backend handles this") so you can focus on their mobile expertise.

### Being Toxic

- **Respectful Demeanor:** Treat the candidate with respect and create a positive and supportive environment for them to showcase their skills.
- **Avoid Belittling:** Avoid making demeaning or condescending remarks, even if the candidate is struggling to answer a question.

### Being Unresponsive

- **Active Listening:** Pay close attention to the candidate's explanations, ask clarifying questions, and provide constructive feedback.
- **Engagement:** Show genuine interest in the candidate's ideas and demonstrate that you're actively engaged in the conversation.

### Being Too Engaged

- **Let the Candidate Lead:** Allow the candidate to lead the discussion and explore different aspects of the design on their own.
- **Provide Guidance, Not Solutions:** Provide gentle guidance and hints when the candidate is stuck, but avoid giving away the answer or forcing them into a specific solution.

### "Grilling" the Candidate

- **Relevant Questions:** Focus on asking questions that are directly relevant to the job requirements and assess the candidate's ability to solve real-world problems.
- **Avoid Gotcha Questions:** Avoid asking obscure or trivial questions (e.g., specific syntax or method names) that are designed to trip up the candidate.

### Asking Meaningless Questions

- **Evaluate Core Skills:** Focus on questions that assess the candidate's understanding of core system design principles, such as scalability, reliability, and security.
- **Avoid Redundant Questions:** Avoid asking questions that have obvious answers or don't provide any meaningful insights into the candidate's abilities.
- **Focus on Decisions:** Ask "Why?" constantly. "Why GraphQL over REST?" "Why Image sizes in the URL?" "Why a Repository pattern here?"

### Not Moving On When the Candidate is Stuck

- **Provide Hints:** If the candidate is stuck on a particular problem, provide a hint or suggestion to help them get unstuck.
- **Shift Focus:** If the candidate is unable to make progress after several attempts, move on to a different topic to avoid wasting time.

### Forcing the Candidate into a Solution They Have in Mind

- **Open to Alternatives:** Be open to alternative solutions and creative approaches, even if they differ from your own preferred solution.
- **Evaluate Based on Rationale:** Evaluate the candidate based on the rationale behind their decisions, not just on whether they arrived at the "correct" answer.

### Digging Too Much into Details (BFS Approach)

- **Prioritize High-Level Understanding:** Focus on assessing the candidate's understanding of the overall system architecture and key design principles, rather than getting bogged down in implementation details.
- **Balance Depth and Breadth:** Strike a balance between exploring specific topics in detail and covering a broad range of topics relevant to the system design.
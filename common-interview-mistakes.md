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
    - **Realistic Scenarios:**  Simulate real-world scenarios with your friends. This could include designing a chat application, a news feed, or an e-commerce platform. Ensure you cover areas like offline support, data synchronization, and push notifications.
    - **Platform-Specific Concerns:** In iOS mock interviews, consider questions around memory management (reference cycles), UI responsiveness, background execution limits, and Core Data/Realm integration. For Android, address issues like activity lifecycle management, background services, battery optimization, and handling different screen sizes and densities.
    - **Time Management:** System design questions can be broad.  Practice timeboxing each part of the discussion (requirements gathering, high-level design, component breakdown, etc.).
    - **Communication is Key:** The mock interviewer should provide feedback not only on your technical solution but also on your communication skills, clarity, and ability to explain complex concepts.

### Rushing Towards a Solution

Take a step back and think!

- **Not Gathering System Requirements:**
    - **User Base:**  "How many users will the app support initially? What is the expected growth?"
    - **Platform:** "Are we targeting only iOS or Android, or both?  What are the minimum supported OS versions?"
    - **Features:** "Which features are core to the app's functionality? Which are optional or can be deferred?"
    - **Non-Functional Requirements:**  "What are the key non-functional requirements like performance, security, scalability, and reliability?  Are there any specific privacy concerns?"
    - **Example:** Don't assume "build a chat app."  Ask about:  Message volume?  Media support? Group chats? End-to-end encryption? Offline access?

- **Not Asking Clarifying Questions:**
    - **Target Market:** Understand the geographical location, user demographics, and network conditions of your target market. This will influence decisions around data storage, CDN usage, and localization.
    - **Audience Size:**  "Is this for a small team, a large enterprise, or the general public?" The scale of the audience will impact the architectural decisions around backend infrastructure and scalability.
    - **Dev Team Details:** "What is the size and expertise of the development team? Are there any specific technology preferences or constraints?" This will affect choices around technology stacks, development tools, and architectural patterns.
    - **Device Capabilities:** "What is the minimum device capability we want to support? This will determine the memory available, processing power and supported OS features. It is important for memory management and performance optimizations."
    - **Example:** "What kind of data are we storing? Is it sensitive personal data that requires encryption at rest?"

- **Jumping Straight to Implementation:**
    - **High-Level Design First:** Start with a high-level overview of the system architecture, outlining the key components, their responsibilities, and how they interact with each other.
    - **UI Architecture Patterns:** Delay discussing specific view classes or UI architecture patterns (MVC, MVVM, VIPER, MVI) until you've established the overall system design.
    - **Example:** Instead of immediately discussing `UITableView` or `RecyclerView`, talk about the overall data flow, data models, networking layer, and caching strategy.
    - **Vendor-Specific Solutions:** Avoid mentioning vendor-specific solutions at this stage. For example, don't say "We'll use Firebase for push notifications" before discussing the overall push notification architecture. Discuss the requirements and different potential approaches for achieving the goal.

### Being Unresponsive

Communicate clearly and constantly.

- **Keeping Silence:**
    - **Think Aloud:** Verbally articulate your thought process, even if you're unsure of the solution. This allows the interviewer to understand your approach and provide guidance.
    - **Explain Your Reasoning:** Explain the rationale behind your design decisions, highlighting the trade-offs and alternatives considered.
    - **Example:** Instead of silently pondering, say, "Okay, I'm thinking about using a message queue for handling background tasks. This would allow us to decouple the UI from the task execution and improve responsiveness. However, it also adds complexity. Let me explain further."

- **Waiting Until the Interviewer Starts Asking Questions:**
    - **Drive the Discussion:** Take initiative by presenting your solution in a structured manner, covering all key aspects of the design.
    - **Anticipate Questions:** Anticipate potential questions from the interviewer and proactively address them in your explanation.
    - **Example:** After presenting the high-level architecture, say, "Now, I'd like to dive into the data synchronization strategy and discuss how we can handle conflicts. This is an area where we need to consider different approaches based on the requirements."

### Giving Up Early

Don't let a single roadblock derail you.

- **Persistence is Key:** System design interviews are challenging. If you encounter a problem, don't give up. Try to identify alternative approaches or break down the problem into smaller, more manageable parts.
- **Focus on the Process:** Even if you don't arrive at the perfect solution, demonstrate your problem-solving skills, critical thinking, and ability to learn from mistakes.

### Talking Too Much

Be concise and relevant.

- **Long Introduction:**
    - **Focus on Relevance:** Keep your introduction brief and focus on the experiences that are directly relevant to the role and the interview question.
    - **Highlight Key Achievements:** Highlight key achievements and projects that demonstrate your expertise in mobile system design, architecture, and development.

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

### Not Explaining Decisions or Not Suggesting Alternatives

Show your thought process.

- **Provide Justification:** Always explain the reasoning behind your design decisions, highlighting the trade-offs and alternatives considered.
- **Consider Alternatives:** For each key decision, briefly discuss alternative approaches and explain why you chose the specific solution you did. This shows that you understand the problem space and have considered different options.

### Lying

Honesty is always the best policy.

- **Be Truthful:** Never exaggerate your skills or experience. If you don't know the answer to a question, it's better to admit it than to make something up.
- **Focus on What You Know:** Focus on highlighting your strengths and areas of expertise, rather than trying to cover up your weaknesses.

### Not Having Any Structure for the Solution

Organize your thoughts.

- **Prepare an Outline:** Before the interview, prepare a general outline of the topics you want to cover, such as requirements gathering, high-level design, component breakdown, and trade-offs.
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
- **iOS & Android Specifics:** For mobile system design, you *must* understand the OS specific constraints and APIs to realistically assess a candidate.

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
- **Avoid Gotcha Questions:** Avoid asking obscure or trivial questions that are designed to trip up the candidate.

### Asking Meaningless Questions

- **Evaluate Core Skills:** Focus on questions that assess the candidate's understanding of core system design principles, such as scalability, reliability, and security.
- **Avoid Redundant Questions:** Avoid asking questions that have obvious answers or don't provide any meaningful insights into the candidate's abilities.

### Not Moving On When the Candidate is Stuck

- **Provide Hints:** If the candidate is stuck on a particular problem, provide a hint or suggestion to help them get unstuck.
- **Shift Focus:** If the candidate is unable to make progress after several attempts, move on to a different topic to avoid wasting time.

### Forcing the Candidate into a Solution They Have in Mind

- **Open to Alternatives:** Be open to alternative solutions and creative approaches, even if they differ from your own preferred solution.
- **Evaluate Based on Rationale:** Evaluate the candidate based on the rationale behind their decisions, not just on whether they arrived at the "correct" answer.

### Digging Too Much into Details (BFS Approach)

- **Prioritize High-Level Understanding:** Focus on assessing the candidate's understanding of the overall system architecture and key design principles, rather than getting bogged down in implementation details.
- **Balance Depth and Breadth:** Strike a balance between exploring specific topics in detail and covering a broad range of topics relevant to the system design.

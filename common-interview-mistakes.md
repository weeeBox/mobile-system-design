# Common Interview Mistakes

## As a candidate

### Not being prepared
- **Study open-source projects** - not every project is perfect but you can still learn lots of useful things from each of them.
- **Study company dev blog** - you can learn a lot about their underlying technical stack and think about potential questions they might ask.
- **Conduct mock interviews with your friends** - time management is crucial: your goal is to cover as much ground as possible in the shortest amount of time. Having some practice also eases up the stress from the actual interview.

### Rushing towards a solution
- **Not gathering system requirements** - the interview question may be purposely vague. The candidate is expected to ask more questions to define the task better. For more information see [Gathering Requirements](https://github.com/weeeBox/mobile-system-design#gathering-requirements).
- **Not asking clarifying questions** - it might be useful to get some information about the target market, the size of the audience, and the dev team details.
- **Jumping straight to implementation** - it's generally a bad sign if the candidate immediately starts discussing low-level topics. For example, which view classes to use or what UI-architecture pattern to apply. The interviewer might not be interested in implementation details in the first place.

### Being unresponsive
- **Keeping silence** - make sure to talk through your solution.
- **Waiting until the interviewer starts asking questions** - it is preferable for the candidate to "drive" the discussion (especially for more senior candidates).

### Giving up early
A single failure might de-rail the candidate and make them give up the whole interview. This is a poor strategy since most of the interviewers are trying to evaluate the candidate's abilities and not only looking for "correct" answers.

### Talking too much
- **Long introduction** - digging deeper into your professional background does not provide much "signal". The interviewer would most likely form an opinion based on the candidate's interview performance and not the summary of their professional experience.
- **Trying to force an interviewing framework** - the said guide represents a set of recommendations and does not define an official interview protocol for any company. The candidate should follow the interviewer's lead in each particular case.
- **Ignoring the interviewer** - it is better to stop talking if the interviewer interrupts you: do not try "to finish the thought" - it is better to move to another topic and not waste time.
- **Repeating yourself** - talking about the same things again: it does not provide any additional information to the interviewer.
- **Jumping from a topic to a topic** - frequent context switching might be hard to follow for the interviewer.
- **Going too broad with the answers** - covering irrelevant things does not provide meaning full "signal" to the interviewer. Try to stick to the original question.

### Treating the interviewer as an adversary
The primary role of the interviewer is to determine if the candidate is fit for the job. If the candidate feels any kind of hostility from the interviewer - it should be reported to the recruiting specialist.

### Trying to fit a solution into an existing scheme
Most pronounced for the candidates who learn particular designs on the Internet and attempt fitting the questions into an unrelated solution.

### Being toxic
- **Being opinionated** - holding a strong belief about certain technology and rejecting any alternative approaches.
- **Interrupting the interviewer** - the interviewer might provide some useful hints or suggest a direction toward the solution. The candidate should let the interviewer finish their thought before bringing up any kind of disagreement.
- **"Educating" the interviewer** - if the candidate believes that the interviewer is wrong/incorrect - it's better to politely suggest this instead of lecturing them on their ignorance. Spending time on "educating" the interviewer provides no useful "signal" and may raise some red flags.
- **Being a "pleaser"** - trying to compliment the interviewer hoping for a positive feedback report in return.

### Not explaining decisions or not suggesting alternatives
It is a red flag when the candidate offers a concrete solution right away without any prior justification. The interviewer might not be able to understand why the candidate prefers a specific approach without comparing it to possible alternatives.

### Lying
Making up things and lying about them is risky since there's a chance that the interviewer might be a domain expert in the area of the discussion.

### Not having any structure for the solution
Most pronounced for the candidates who don't prepare for their rounds. Not having a plan for structuring the solution would put additional pressure during the interview.

### Speaking in terms of specific vendors
Using vendor-specific solutions during the interview makes it more about the implementation details and not about architecture design. The candidate may omit all vendor information unless the interviewer explicitly asks about it.

### Being overly optimistic
Sometimes, the candidate does not take the "unhappy" path into account. For example, running out of memory or disk space.

### Making assumptions
It's better to admit that you don't know how things work instead of making assumptions. It's ok to reason about how a certain functionality _could_ be implemented but you need to state your lack of knowledge first.

### Ignoring the interviewer's hints
The interviewer is there to help you - take a hint!

## As an interviewer
_Note: Cognitive hiring biases are left out of scope for this guide. You can [find](https://blog.staffingadvisors.com/5-cognitive-biases-that-get-in-the-way-of-hiring) lots of information on the Internet._

### Not being prepared
The interviewer should carefully think through the question and figure out possible topics for discussion and follow-up questions. The selected topics should reflect the candidate's strengths and not the interviewer's experience.

### Being toxic
- **Being a jerk** - treat the candidate with disrespect.
- **Acting "superior"** - showing your expertise and trying to belittle the candidate.
- **"Educating" the candidate** - some interviewers may ask a question and then answer it themselves when the candidate fails to do so. It does not help to evaluate the candidate and wastes the interview time.

### Being unresponsive
- **Not paying full attention** - checking the phone, daydreaming, or responding to emails.
- **Eating** - no comments.

### Being too engaged
- **Suggesting solution to the candidate** - pushing a candidate in the "right" direction, giving too many hints.

### "Grilling" the candidate
- **Asking narrow sub-domain questions** - a type of questions focusing on specific things the candidate does not need to know to be successful at the job.
- **Trying to corner the candidate** - asking intentionally tough and tricky questions to make the candidate admit they don't know the answer.

### Asking meaningless questions
- "Why did you choose a relational database (ORM) to store structured data that requires complex querying?" - there's little "signal" a candidate can provide here (unless they advocate for text files or user preferences).
- "Why caching/modularization/testing/etc is necessary?"
- "Why would you use an industry-standard library instead of a custom implementation?" - this question might make sense when it comes to framework development but _could_ be meaningless when talking about an app. See "Chapter 21. Dependency Management" from the "Software Engineering at Google" book.

Everything the interviewer asks should help determine if the candidate is a fit for the job. Any other questions should be left out of scope.

### Not moving on when the candidate is stuck
Letting the candidate "struggle a little" and allowing long pauses. It's better to give a hint sooner than later or move to the next topic.

### Forcing the candidate into a solution they have in mind
It won't help the candidate to show off their strong side.

### Digging too much into details (BFS approach)
Overly concentrating on a narrow topic and leaving high-level stuff behind.

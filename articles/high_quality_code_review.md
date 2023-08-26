## Overview

In this article, we are going to declare the structure of a code review process that will help you build a product with better code quality in less time. We will begin by explaining the main reasons for code reviews and the benefits they can provide to your team. Next, we will identify the key participants in code reviews, their responsibilities, and the workflow. After that, we will delve into a detailed discussion of the plans for the organization, code authors, and reviewers that will help them get the most out of code reviews and achieve excellent results.

## Why do we need code review?

Code review is a process where someone other than the author(s) of a piece of code examines that code. It is mostly used for the following reasons:

- **Minimizing mistakes and their impacts:** 

> This includes making sure bugs and defects are prevented as much as possible and that the source code is of high quality.

- **Maintaining norms:**

> This involves ensuring that there are adequate tests, consistency in style, design, and implementation.

- **Meeting requirements:** 

> Based on Acceptance Criteria.

- **Improving code quality/performance/security.**

- **Learning and sharing:** 

> Both domain and coding knowledge.

- **Tracing and tracking decisions:** 

> Understanding the evolution of the code and why and how changes have happened.

- **Gatekeeping:** 

> Ensuring security and having an additional safety net so that a single developer cannot commit arbitrary code.

- **Finding Alternative Solutions.**

## Structure Overview

### Actors

First of all, let's define the main actors of code review. I would choose the following:

- **Organization**
- **Code Author**
- **Core Reviewer**

### Responsibilities

Let's define their responsibilities:

**Organization**

The organization's responsibility is to simplify life for Code Authors and Reviewers by defining rules and processes. Every question related to the code review process should have answers provided by the organization. Here are examples of questions that can be raised and should have answers:

- How to encourage team members to participate in code reviews?
- How to resolve conflicts?
- How to determine when a review is complete and it's okay to approve?
- How to encourage team members to follow the process?
- What to do if you have a strict deadline or need to merge a critical bug fix as soon as possible?
- How to respond when someone says 'No' during the review?
- Who will perform code reviews?
- How often should code reviews take place?
- Who is responsible for assigning tickets/code reviews to relevant engineers?
- What should you do if no one wants to review your PR?
- etc.

**Code Author**

The Code Author's main responsibility is to implement a high-quality solution and to simplify the life of the reviewer as much as possible.

**Code Reviewer**

The Code Reviewer's main responsibility is to double-check the author's work and identify improvements that the author may not have noticed.

### Flows

Let's define the flow for each of them:

**Organization**

The Organization should create a document with answers to all possible questions that may arise and ensure that teams adhere to the established rules.

**Code Author**

There are 5 points that every Code Author should follow:

1. **Create a Pull Request (PR) and describe the changes to provide more context for reviewers.**
2. **Perform a self-check and review their code before submitting it.**
3. **Find the best reviewer.**
4. **Collaborate with the Reviewer to resolve all issues and obtain approval.**
5. **Follow the organization's process to merge the changes.**

**Code Reviewer**

There are 5 points that every Code Reviewer should check before giving approval:

1. **Ensure that the code follows business requirements.**
2. **Ensure that complexity has been reduced.**
3. **Ensure that the design is good.**
4. **Ensure that reliability is at the appropriate level.**
5. **Check that the code doesn't have any hidden risks.**

In addition, the reviewer should know how to collaborate with the author to resolve all issues.

Let's discuss these responsibilities and flows for each actor more precisely.

## The Standard of Code Review

### Organization-Wide Practices

During code reviews, many issues can arise, and we need to know how to handle them. To address these problems and ensure that everyone is on the same page and at the same position, the organization should create a document outlining engineering practices and rules to document and standardize the code review process. This document should cover the following main points:

- Ensure that the purpose of code reviews is clear to everyone.
- Ensure that the "Definition of Done" is documented and clear to everyone.

> This is an essential point because we need to establish boundaries. We can't attain perfection, so we need to define minimum rules required for a PR to be considered good enough. For example, the following criteria could be considered:
> 
> - Test coverage is sufficient and correct; all border cases are covered.
> - CI is green, and all tests have passed.
> - Rubocop and other style linters have passed.
> - Etc.

- Encourage team members to actively participate in code reviews.
- Define a process for conflict resolution during code reviews.

> P.S. You can read Google's process for conflict resolution here: [Resolving Conflicts](https://github.com/google/eng-practices/blob/master/review/reviewer/standard.md#resolving-conflicts-conflicts).

- Set clear expectations for code review turnaround times.
- Prioritize code reviews as an essential task.
- Leverage code reviews as opportunities for knowledge sharing and learning.
- Encourage reviewing code in unfamiliar areas to promote cross-functional knowledge.
- Continuously monitor and improve the code review process.
- Recognize and reward those with a track record of providing quality feedback.
- Conduct regular code review sessions to discuss broader trends or issues that arise during the review process.
- Encourage authors to seek feedback during development before submitting a formal code review.
- Ensure that you have proper processes in place to speed up reviews.

> P.S. Learn more about how Google accelerates reviews here: [Fast Review](https://github.com/google/eng-practices/blob/master/review/reviewer/speed.md).

- Determine how to handle emergencies

> P.S. Learn more about how Google handles emergencies here: [Emergencies](https://github.com/google/eng-practices/blob/master/review/emergencies.md).

Here's the full document from the Google team outlining their practices: [Google Engineering Practices Documentation](https://github.com/google/eng-practices).


## Code Author

Your goal as an author of code seeking review is to simplify the reviewer's life as much as possible. You are the one in need of the review and who should receive the review, not the reviewer who wishes to spend their time reviewing your code. Therefore, ensure that you do not compel them to perform tasks that you could handle yourself.

There are 5 steps that every Code Author should follow:

1. **Create a Pull Request (PR) and describe the changes to provide more context for reviewers.**
2. **Perform a self-check and review your code before submitting it.**
3. **Find the best reviewer.**
4. **Collaborate with the Reviewer to resolve all issues and obtain approval.**
5. **Follow the organization's process to merge the changes.**

### Providing Context for Reviewers

- **Ensure that you include a proper title, description, any screenshots, relevant links, configuration changes, etc., in the PR.**

> P.S. You can read Google's requirements for the description here: [Google PR Description Guidelines](https://github.com/google/eng-practices/blob/master/review/developer/cl-descriptions.md).

- **Make sure to add a very clear reason for why you're making these changes**

> Achieving code perfection is challenging because every code change introduces a risk of breaking something. Without code, there's no risk. Therefore, it's crucial to ensure that there are no alternative solutions to the problem. If you only describe the code changes without providing the reason, you're depriving the reviewer of the opportunity to suggest a completely different solution to the problem. While the reason may be clear to you, keep in mind that reviewers may not have your context, so it's essential to add it.
>
> Even if you and your reviewers currently understand the reason, consider adding it for historical purposes. In a few years, you may forget the initial rationale, making it much harder to replace or refactor the solution. For example, if someone later introduces a new global solution that could solve the problem more efficiently, they might struggle to replace the code because they expected your code to perform additional functions.

- Take notes on any questions or concerns about the change to discuss them during the review.
- Identify any potential performance, security, or scalability concerns and note them for discussion during the review.
- Avoid attempting to sneak something past reviewers; highlight questionable elements instead.
- Consider the impact of the change on other parts of the system and express your concerns to others.
- Provide context for your design choices by explaining why you chose a particular approach.
- If there are any alternative solutions that were considered and why they were not chosen, briefly mention them.

### Self-Check: Review Your Code Before Submitting

- Before submitting a PR, thoroughly test your changes locally to identify and address any obvious issues.
- Write automated/unit tests to validate the functionality.
- Double-check for spelling mistakes.
- Perform code cleanup, removing comments, todos, and unnecessary artifacts.
- Confirm that the code passes all tests, including unit, integration, and system tests.
- Review the code to ensure it aligns with the project's coding standards, team guidelines, and best practices.
- Maintain consistency with the overall project design and architecture.
- Write or update documentation for the changes if required.
- Keep your code changes concise and focused. Avoid overwhelming reviewers with extensive code in a single review session.

> It has been proven that we get more value from code reviews when the PR is smaller rather than big. If you are struggling to keep them small, try pairing with a teammate (or engineers from other teams).
>
> P.S. You can read about why Google prefers to keep PRs small here: [Google Engineering Practices](https://github.com/google/eng-practices/blob/master/review/developer/small-cls.md).

### Finding the Best Reviewer

- **Who to Choose:**
  - Someone capable of responding to your review within a reasonable period of time.
  - Someone who can provide the most thorough and accurate review for the piece of code you are writing.
  - Look for reviewers who have expertise in the domain or specific functionality related to your code changes.
  - Reviewers who have been involved in previous discussions or decisions related to the code you are changing may have valuable historical context.

- **Tools to find:**
  - Check the codeowners file.
  - Use `git blame` to identify potential reviewers.
  - Document which teams are responsible for specific areas to facilitate reviewer selection.

### Collaboration

- Approach the review process with an open mind and be willing to learn from and collaborate with other team members.
- Address all the feedback received, including any concerns or questions raised.
- Don't dismiss or resolve a comment without explanation. Every comment is valid; as a code author, even if you feel the comment isn't relevant, please address it so the reviewer can understand the context and why you've done something in a certain way.
- Seek feedback from other team members if you are unsure about the changes.
- If you're unsure about a review comment or suggestion, don't hesitate to ask for clarification.
- Don't take it personally. It's the code under review, not you.
- View each review as a chance to learn and grow as a developer. Embrace feedback as a path to improvement.
- Be open to new suggested approaches, even if you consider them not too important; it's a good opportunity to learn a new way of doing things.
- Do not make every change that is suggested if you still believe your approach is better. Have a conversation and agree with the reviewer if possible.
- Treat everyone's opinion the same, regardless of seniority. Ensure you don't apply a change if someone senior told you to do so without understanding the reasons.
- Take ownership of your code and its quality. Be accountable for the changes you make.
- Ensure that all code review discussions, comments, and decisions are documented within the code review platform. This creates a historical record and helps other team members understand the rationale behind the code changes.
- Create a positive and collaborative atmosphere by expressing your gratitude to the reviewer

> Good: 'Thank you!'
>
> Good: 'Nice catch!'
>
> Good: 'Thanks for pointing that out'
>
> Good: 'Thanks for taking the time to review'

### After Review  

- **Express Gratitude**

> Code reviews are, in large part, about having others watch your back. Don't hesitate to express your gratitude with a simple "Thank you" once the review is completed. If you're new to code reviews, take a moment to reflect on what went well and what didn't.

- **Approval Checks:**
  - Ensure that you have received enough approvals for your code changes.
  - If you've made changes to unfamiliar areas, ensure you've received approval from the code owner.
- **Testing and QA:**
  - Ensure that your ticket will be tested by Quality Assurance (QA).
  - Provide clear and comprehensive descriptions of all places you've modified in the ticket description.
- **Continuous Integration (CI):**
  - Verify that the CI pipeline has successfully passed for your code changes.
  - Confirm that the main branch's CI pipeline is not failing.
- **Confirm that it's an appropriate time to merge your changes:**
    - Not too late or too early to ensure support if issues arise.
    - Not during holidays
    - Ensure that you do not merge code during the process of fixing critical issues to avoid additional problems.
- **Merge the approved code changes into the main branch.**
- **Post-Merge Checks:**
  - Ensure that the main branch's CI pipeline remains green after merging your changes.
  - Verify that the code changes are functioning as expected in the production environment.
  - Keep an eye on monitoring tools to promptly address any issues that may arise due to your code changes.
  - Review and update documentation, such as README files, API documentation, or user guides, to reflect any changes made in the code.
- **Celebrate**

## Reviewer

The main goal and priority of a review are:

- **Reduce the risk of breaking existing logic and introducing new bugs.**
- **Avoid complicating the lives of other programmers in the future.**

We can split the reviewer's task into two parts:

- **Hard Skills (What Do Code Reviewers Look For?):**
  - Meeting business requirements (based on Acceptance Criteria)
  - Reduce Complexity(Style, Consistency, Readability)
  - Improve Design (Maintainability/Testability/Stability/Reusability)
  - Improve Reliability (Performance/Security)
  - Mitigate Risks
- **Soft Skills (How should Code Reviewers behave to address the problems they find?):**
  - Collaboration
  - Respectful Comments

So, if your code doesn't break anything and simplifies the lives of other developers, it's okay to approve it. Below, we will discuss the main points on how to achieve this.

## Hard Skills (What Do Code Reviewers Look For?) 

Code reviewers should consider the following points:

- **Meeting business requirements**
- **Reduce Complexity**
- **Improve Design**
- **Improve Reliability**
- **Mitigate Risks**

### Meeting business requirements

If the author hasn't implemented what should be done according to the business requirements, then all other reviews will be a waste of time. So, your priority here is to identify this issue as soon as possible.

- **Review any documentation or design specifications related to the change (JIRA story, design document, etc.) and compare them with the author's description of the changes.**
- **Based on the requirements, prepare a list of items that should have been covered in the changes.**
- **Review Functionality Based on Acceptance Criteria**
  - Do we really need it? Does the change make sense? Does it have a good description?
  - What problem do we want to solve?
  - Is the business logic correct?
  - Are the Acceptance Criteria satisfied?
  - Are there any potential edge cases or scenarios not covered by the acceptance criteria that should be considered?
  - Does the code behave as the author likely intended?
- **Review Usability and User Experience**
  - Are error messages clear and user-friendly?
  - Does the UI gracefully handle unexpected errors or input?
  - Does the UI support multiple languages and locales?
  - Does the UI provide a good user experience on mobile devices?

### Reduce Complexity 

If you can't understand the code, it's very likely that other developers won't either. So you're also helping future developers understand this code when you ask the developer to clarify it.

"Too complex" usually means "can't be understood quickly by code readers." It can also mean "developers are likely to introduce bugs when they try to call or modify this code."

- **Complexity**
  - Could the code be made simpler? 
  - Can the solution be simplified?
  - Does similar functionality already exist in the codebase? If yes, why isn’t it reused?
  - Is the PR more complex than it should be? Could it be split into parts?
  - Could some parts be moved to another ticket?
  - Are there redundant or outdated comments? Is there any commented-out code?
  - Is the developer not implementing things they might need in the future but don't need now?
- **Readability**
  - Would another developer be able to easily understand and use this code when they come across it in the future?
  - Does your code tell the story clearly? Or does it clearly explain the business flow?
  - Did the developer choose clear names for variables, classes, methods, etc.?
  - Are there excessive abbreviations or cryptic variable names that could be made more descriptive?
- **Style**
  - Does the code follow the company-defined style guides?
  - Is code formatting consistent, including indentation, spacing, and line breaks?
- **Consistency**
  - Is there a consistent approach to coding within your company's rules?
  - Does the code change adhere to the project's coding standards and best practices?
  - Are naming conventions consistent with the project's coding standards?

### Improve Design

In fact, reviewing the rest of the PR might be a waste of time because if the design problems are significant enough, a lot of the other code under review is going to disappear and not matter anyway.

- **Design**
  - Is the code well-designed and appropriate for your system?
  - Are there any best practices, design patterns, or language-specific patterns that could substantially improve this code?
  - Can we improve the code via design patterns?
  - Does this code adhere to Design Principles? (KISS, SOLID, YAGNI, etc.)
  - Have you found any code smells in the code?
- **Is the code well-designed?**
- **Will it be easy to change the code for other developers in the future?**
- **Are architectural or major design changes approved by a Principal or an Architect?**
- **Are you sure that no new unit/gem/architecture/etc. is introduced into the codebase without the prior approval process (tech leadership meeting) or at least being discussed with a Principal?**

### Improve Reliability

This step allows you to identify any potential performance, security, or scalability concerns and discuss them with the author.

- **Perfomance** 
  - Do you think this code change will negatively impact system performance?
  - Do you see potential for significant code performance improvement?
  - Are database queries optimized for efficiency, with proper use of indexes and joins?
  - Are N+1 query problems addressed by using eager loading where needed?
  - Are computationally expensive tasks offloaded to background jobs, preventing slow responses or request timeouts for users?
  - Does this code utilize memoization for methods that are called multiple times?
- **Security**
  - Which user roles are expected to gain access to the added resources?
  - Are passwords securely stored using strong encryption, such as bcrypt?
  - Are file uploads properly validated, restricted to safe file types, and securely stored?
  - Does this code change expose any sensitive information, such as API keys, passwords, or usernames?
  - Is sensitive data, like user data or credit card information, securely handled and stored?
  - Are user inputs validated and sanitized to prevent SQL injection and XSS attacks?
  - Are authorization and authentication correctly implemented? 
  - Are third-party libraries and gems regularly reviewed for known security vulnerabilities?
  - Are API endpoints authenticated and authorized correctly?
  - Are you sure that users cannot access API resources that do not belong to them?

### Risk Management 

The main goal of this step is to mitigate the risk and do everything you can to reduce the chances of breaking the product.

- Preemptively draw attention to any areas of the code where you feel unsure.
- Determine the appropriate level of review needed based on the scope and impact of the code change.
- How risky is the PR? Is the feature actively used, or is it hidden and hasn't been realized yet? How critical is it?
- Is there any impact on other domains? Could the owner of the PR lack knowledge about how their work might impact other domains or code?
- How experienced is the PR author? Do you need to spend extra time re-checking the author's code, or can you trust it?
- Make a list of potential risks or issues that could arise from this change.
- It might be helpful to try running the code locally to ensure there are no issues.
- **Tests**
  - Review any tests included with the code change to verify they adequately cover the functionality and edge cases.
  - Is test coverage sufficient and correct? Are all border cases covered?
  - Do the code changes have appropriate unit tests?
  - Is the code designed to be easily testable?
  - Have automated tests been added or relevant ones updated to cover the new functionality?
  - Do the existing tests reasonably cover the code change, including unit, integration, and system tests?
  - Are there additional test cases, inputs, or edge cases that should be considered?
- **Logic Errors and Bugs**
  - Can you envision any use cases in which the code does not behave as intended?
  - Can you identify any inputs or external events that could potentially break the code?
  - Ensure a failing test is written if the change is for a bug fix.

## Soft Skills (How should Code Reviewers behave to address the problems they find?)

### Collaboration

- Be willing to collaborate with the author to resolve any issues or concerns that arise during the review process.
- Seek continuous improvement, not perfection.
- Consider using pair programming as an alternative or supplement to code reviews.
- Resolve conflicting opinions in a timely manner; don't let a PR sit around due to disagreement.
- Allocate time for code reviews - everyone really appreciates reviews. You get to know the people and the codebase too.
- Articulate your intention behind the feedback (suggestion vs request for a change vs improvement that can be done in a different ticket).
- Keep all the discussion online. If you contacted the reviewer by chat or email, bring relevant comments online.
- Be kind and empathetic.
- Reply within a reasonable timeframe.
- Avoid Gold Plating. Yes, we strive for code excellence but at some point, we have to be pragmatic (create a refactor ticket) and ship code.
- If you review only certain files that are part of a larger change or only certain aspects of the PR, such as the high-level design, privacy or security implications, etc., note in a comment what you reviewed.
- Discuss in person

> If there is a disagreement, have a quick in-person/video/IM chat to sort out what is going on - it‘s much easier to address all the little "Oh, I didn’t know"s in a single face-to-face, instead of back-and-forth via e-mail with long delays. “In person” doubly applies if you are disagreeing with another reviewer. And please make sure to record the outcomes on the review.

- Find an end

> If you like things neat, it‘s tempting to go over a code review over and over until it’s perfect, dragging it out for longer than necessary. It‘s soul-deadening for the recipient, though. Keep in mind that “LGTM” does not mean “I vouch my immortal soul this will never fail”, but “looks good to me”. If it looks good, move on. (That doesn’t mean you shouldn‘t be thorough. It’s a judgment call.) And if there are bigger refactorings to be done, move them to a new PR.
- Always ask yourself if this decision really matters in the long run or if you're enforcing a subjective preference.

### How to provide respectful and professional feedback?

- **Use "we" instead of "you"**

> **Bad**: *"Why do you add this function?"*
>
> **Good**: *"Why do you think we should add this function?"*

- **Use questions instead of commands or orders**

> **Bad**: *"We need tests for this."*
>
> **Good**: *"I don't see any unit tests for this function. Could we add some test cases to ensure it behaves as expected under various conditions?"*

- **Suggest, rather than instruct**

> **Bad**: *"Change the variable name to 'total_price'."*
>
> **Good**: *"Consider changing the variable name to 'total_price'. It might make the code more descriptive."*
> 
> **Good**: *"Perhaps it would be better to rename the variable as 'total_price' for improved code clarity."*

- **If you add something not very important, feel free to prefix it with something like 'Nit' to let the author know that it's just a point of polish that they could choose to ignore**

> **Bad**: *"The formatting here is inconsistent."*
>
> **Good**: *"Nit: I noticed a minor inconsistency in the formatting. Could we align the indentation for consistency?"*
> 
> **Good**: *"Optional: I noticed a minor inconsistency in the formatting. Could we align the indentation for consistency?"*
>
> **Good**: *"FYI: I noticed a minor inconsistency in the formatting. Aligning the indentation for consistency is something to consider."*
>
> This makes the reviewer's intent explicit and helps authors prioritize the importance of different comments. It also helps prevent misunderstandings. For instance, without comment labels, authors might interpret all comments as mandatory, even if some are intended to be purely informational or optional.

- **Don’t be afraid to ask questions about how or why a thing works.**

> **Good**: *"Could you provide some context on why you decided to use this specific algorithm instead of a more straightforward approach?"*

- **Keep it light-hearted and use emojis**

> **Good**: *"What do you think about this approach? 🤔 But I'm not sure if it works 😅"*

- **Try to find positives and provide positive remarks**

> **Good**: *"Great work on this feature! 🚀"*
>
> **Good**: *"Excellent job! ✅"*
>
> **Good**: *"Well done 💪"*
>
> **Good**: *"Thanks for fixing it so quickly 🎉"*

- **Raise concerns in a polite and constructive manner**

> **Bad**: *"This is wrong"*
> 
> **Good**: *"Maybe I'm missing something, but ..."*
>
> **Good**: *"I might be mistaken, but ...?*
>
> **Good**: *"I could be wrong, but ..."*

- **Be respectful and professional in your feedback, avoiding personal attacks, blame, or derogatory comments**

> **Bad**: *"This code is so sloppy and careless. Are you even trying?"*
>
> **Bad**: *"You're making the same mistake again."*
>
> **Bad**: *"This code is terrible."*
>
> **Bad**: *"You clearly don't understand how this works."*
>
> **Bad**: *"You always forget to add comments."*
>
> **Bad**: *"How could you not see this mistake? It's so obvious."*
> 
> **Bad**: *"Are you sure that software engineering is the right career path for you?"*

## Conclusion

To wrap it up, this article has introduced a clear code review process that can enhance code quality and speed up product development. We've talked about why code reviews matter and how they help teams. We've also pointed out who does what in code reviews and how it all works. These ideas are here to support organizations, code authors, and reviewers in making the most of code reviews and getting excellent outcomes.

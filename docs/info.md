

Thanks for sharing this — the overall direction of a low-impact transition with dual deployment is a good approach to reduce risk during migration.

I do have a few observations and questions from an architecture and operational standpoint that we should align on before scaling this further:

1. CI/CD Boundary Clarity
The current approach introduces additional CI pipeline logic (scripts, Java-based control) to manage what is fundamentally a CD concern (ArgoCD-driven state reconciliation).

We should be careful not to couple deployment orchestration into CI, as ArgoCD is designed to operate declaratively from Git as the source of truth. Introducing imperative logic in CI could lead to drift and reduce transparency.

2. Source of Truth vs Control Flow
While the note mentions Git (values.yaml) as the source of truth, the pipeline is programmatically updating branches and triggering deployments.

It would be good to clarify:

Are we moving toward a fully GitOps-driven model, or
Maintaining a hybrid CI-triggered deployment model?

This distinction is important for long-term operability and auditability.

3. Dual Deployment Strategy (Rafay + ArgoCD)
The “dual control / kill switch” approach is useful short-term, but we should define:

Clear ownership of deployment state during transition
Exit criteria for Rafay to avoid prolonged dual control complexity

4. Deployment Abstraction in Java Layer
Centralizing deployment logic in Java for propagation across repos is interesting, but it introduces an additional abstraction layer outside native GitOps tooling.

We should evaluate whether this creates tight coupling or reduces visibility for teams operating the platform.

5. Operational Model & Failure Handling
Since scripts are modifying deployment artifacts and pushing changes:

How are failures handled (partial commits, rollback, drift detection)?
How does Argo reconcile unexpected states introduced via CI?

6. Environment Promotion Flow
The note mentions Dev → QA migration sequencing — it would be helpful to confirm if promotions remain Git-driven (PR-based) or pipeline-triggered.

Aligning this early will help standardize patterns across teams.

Overall, the direction makes sense for a controlled transition, but I think we should align on the target-state model (pure GitOps vs hybrid) and ensure current decisions don’t introduce long-term operational complexity.

Happy to walk through this together and help standardize the pattern.


****

I want to briefly provide additional insight from yesterday JTAPPS calls, as it is starting to impact both delivery and overall program alignment along with my personal life.
I understand this is not the best professional letter that I should be giving an MD but I think my emotions are running high and I am not able to understand what is going on in Jefferies anymore.
During the JTAPPS Daily Scrum. Anish and Shiva both did not act as a SR. VP level individuals. I am not sure if is it their lack of skills or ulterior motives. Consciously I have never been part of any politics ever in my professional career, I have never taken credit for anyone else work.
I come from an Army back ground, hence integrity is the core of our family. So, I will always do what Is morally right. Right for Computer Science, right for technology, right for Jefferies, right for you.
I do not think Shiva has your best interest in his mind. He appears professional but is …. I give up …. a freaking weasel man, I understand being a professional but my definition of profession is deeply rooted with passion and integrity. And I understand business and how it works. For god sake I was part of the Work Wide Strategy and Solutions team at HP, building state of the art systems for the fortune 500!!!!!! But man, Jefferies is a freaking night mare at my level, I have never seen this much politics and cut through place. And I think I know why! 
Even Shiva your most trusted is hiding things from you and not telling you the right things, Amit chahal whom you are very close to is pushing back, is it because he wants on premise to survive as it is a support organization  .. is this normal ?
Everytime I think of Shiva my blood boils .. do you remember the day when I asked him in front of you that I have walked Shiva with all the documentation and knowledge transfer he did not acknowledge and acted like that was nothing. You know what I told him……  all the hidden dynamics of the organization, how the different individuals are, all the data that we collected on IPM and a lot fucking more. I understand being a professional and sensitive with information but just manipulating it all the fucking time… and It is partly your fault you presented Shiva as this amazing person you are bringing to the org, expected more from you …..
It could be because 
1.	He does not understand half of that shit outside his strength which is functional trading knowledge
2.	Which is kinda true ….

You want to know funny incidents 
a.	I said workshop and he started saying workshop 2 week after and now is going around everywhere in front of everyone let’s do workshop lets do a workshop, no credit
b.	I showed will all the workstreams and then he shows all the workstreams to the team, NO credit 
c.	I told him the first time Anish misbehaved .. he said oh its ok
d.	I told him this is such a fucking issue 
I am observant but not dumb, I wanted to come to you but you were not going to trust me as I already lost credibility with you and you were always being quite frankly a jerk. Every meeting was a stressful meeting and I don’t now why!  YO please lets first get on the same page…..
p.s. I do respect you, your knowledge, and see all the things you do but this is ridiculous 

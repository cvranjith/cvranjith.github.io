[The recent case study of Amazon Prime Video](https://www.primevideotech.com/video-streaming/scaling-up-the-prime-video-audio-video-monitoring-service-and-reducing-costs-by-90) showcases an interesting transition from a serverless microservices architecture to a monolithic approach, which resulted in a significant 90% decrease in operating expenses for their monitoring tool. This shift has sparked discussions about the differences between serverless and microservices and how to evaluate their respective advantages and disadvantages. The study revealed that serverless components, including AWS Step Functions and Lambda, were causing scaling bottlenecks and increasing costs. By removing these serverless components and simplifying their architecture, Amazon Prime Video claims to have achieved substantial cost savings.

This shift underscores the importance of striking the right balance between serverless and microservices architectures for specific use cases. While microservice and serverless computing can provide benefits such as scalability and reduced operational overhead, they may not always be the optimal solution for every application or system. Similarly, microservices can offer increased flexibility, but they may also introduce unnecessary complexity in certain situations. Developers must evaluate their project requirements and constraints carefully before deciding which architectural patterns to adopt.

![monolith](https://github.com/cvranjith/cvranjith.github.io/blob/main/imgs/monolith.png?raw=true)

Reference to Amazon Prime Video case study >> [link](https://www.primevideotech.com/video-streaming/scaling-up-the-prime-video-audio-video-monitoring-service-and-reducing-costs-by-90)

>quote:
>"We designed our initial solution as a distributed system using serverless components... In theory, this would allow us to scale each service component independently. However, the way we used some components caused us to hit a hard scaling limit at around 5% of the expected load."


The Video Quality Analysis (VQA) team at Prime Video had a tool for audio/video quality inspection, but it was not designed for high-scale usage, and it became too expensive to operate at high volumes. They identified scaling bottlenecks that prevented them from monitoring thousands of streams, so they reevaluated the architecture to address cost and scaling issues.

<html>
  <table>
    <tr>
      <th>The initial architecture of defect detection system.</th>
      <th>The updated architecture for monitoring a system with all components running inside a single Amazon ECS task.</th>
    </tr>
    <tr><td><img src="https://github.com/cvranjith/cvranjith.github.io/blob/main/imgs/pv-arch-1.png?raw=true"></td>
      <td><img src="https://github.com/cvranjith/cvranjith.github.io/blob/main/imgs/pv-arch-2.png?raw=true"></td>
    </tr>
  </table>
</html>


When it comes to building software applications, there are many different architectural styles to consider. Two popular approaches are monolithic and microservices architectures.

## Monolithic vs Microservice architecture

Monolithic architecture involves a single block of code with multiple modules and separate tiers, while microservices architecture is made up of separate modules called microservices, each with its own database and communicating through APIs and HTTP protocols.


Both monolithic and microservices architectures have their own unique strengths and weaknesses. For instance, a monolithic system can be practical and efficient for building large-scale applications, while microservices can offer increased flexibility and scalability. However, each approach may not be the optimal solution for every application or system.
![monolith-vs-ms](https://github.com/cvranjith/cvranjith.github.io/blob/main/imgs/monolith-vs-ms.png?raw=true)



## Majestic Modular Monoliths

A hybrid approach called "Majestic Modular Monoliths" has emerged, combining the benefits of both monolithic and microservices architectures. This approach involves breaking down the monolith into modules, but keeping them within the same codebase and deployment unit. This allows for improved scalability and maintainability, while avoiding some of the complexity and cost overhead of a fully distributed microservices architecture.


## Domain-Driven Monolith

Another interesting approach is the Domain-Driven Monolith, where each vertical slice represents a distinct subdomain of the overall application's domain. This approach enables developers to focus on delivering business value and adds flexibility to add or remove functionality without disrupting the entire system.

## Striking the balance

The case study of Amazon Prime Video emphasizes the importance of choosing the appropriate combination of serverless and microservices architectures that suits specific use cases. While serverless computing may offer advantages such as scalability and reduced operational overhead, it may not be the optimal solution for all applications or systems. Similarly, microservices can provide increased flexibility, but they may introduce unnecessary complications in certain scenarios. Prior to selecting architectural patterns, developers should meticulously evaluate their project requirements and limitations. The Amazon Prime Video team discovered that eliminating serverless components from their architecture and adopting a monolithic approach led to better optimization of both cost and performance.

![monolith-vs-ms](https://github.com/cvranjith/cvranjith.github.io/blob/main/imgs/striking-balance.png?raw=true)


## Beyond the Buzzwords: Making Informed Decisions

The analysis presents significant knowledge into the practical difficulties and expenses connected with serverless and microservices designs. It accentuates the significance of discerning the disparities between these notions and their appropriateness for diverse situations. Developers can make informed judgments that enhance both cost-effectiveness and efficiency by maintaining an equitable strategy and continually examining the trade-offs between different designs. This research provides a candid view of how to reduce expenses by adopting a streamlined architecture and being open to altering direction.

* Netflix's move to microservices was a blockbuster hit, and paved the way for other companies to follow suit. It's no wonder they're still leading the streaming game!
* Amazon Prime Video's switch to a monolithic architecture may have raised some eyebrows, but it just goes to show that there's no one-size-fits-all solution in the world of software development. It's all about finding the right fit for your specific needs.


In the battle of microservices vs monoliths, both sides have their strengths and weaknesses. It all comes down to your project requirements and constraints - and a healthy dose of trial and error!

One day it's 'I love microservices', the next it's 'I need my monolith back'. It's like they're trying to make their architecture as binge-worthy as their shows :) 



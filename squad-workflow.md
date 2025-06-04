# Asana &amp; ticket management

## Ticket lifecycle

Tickets are expected to enter as problems, they are discovered by the PMs to decide if they worth entering the system.

When accepted they enter into the design step where the team gathers requirements and discuss different solutions to the problem.

Once all the requirements are gathered and a solution is proposed the engineering team will review the ticket to ensure they got all the information required to consider a ticket to be ready for development.

Tickets considered ready for dev will be moved to some sprint where they will be developed. Some status (in progress, in review, merged…) are tracked to facilitate development.

![](https://slabstatic.com/prod/uploads/fcn5vyb4/posts/images/nyq-5Cgj_tEEfaneUMzG7PpE.svg)





We use Asana as our ticket tracking tool where we have [a team](https://app.asana.com/0/1201433513169957/overview) to group all related tasks. Under this team we will create projects using one of our two templates:

![](https://static.slab.com/prod/uploads/fcn5vyb4/posts/images/LIvDl962xb2fAM_wJDRMoVVC.png)

## Backlogs

At first tickets are added to the team backlog. We have 2 backlogs the product backlog where new features and bugfixes are tracked and the tech backlog, where tech debt, library updates and refactors are tracked.

### Product backlog





![](https://static.slab.com/prod/uploads/fcn5vyb4/posts/images/gXkBBtgOo-Z6u3XCoeJvh1cE.png)



Upcoming user stories and tasks will be reflected in the backlog which has the following sections:

- **TODO**:
    - A ticket which contains an idea or a bug report that needs to be explored.
    - It is the Product Manager&#39;s responsibility to review these tickets and add them to relevant Asana projects
    - Typically, requests from stakeholders (ex: Prod Ops or Customer Experience) will appear in this section, at which point the Product Manager picks it up for discovery and research
- **DISCOVERY:** 
    - A ticket that&#39;s being _in discovery -_ This mean that the Product Manager is gathering all the required information, tech implications, UI/UX, legal stuff before is ready to be developed.
    - There may be additional grooming or refinement sessions with the stakeholder who initially submitted the request
    - There may also be times that the Product Manager may have to bring in the Engineering Lead into the discussion to help with discovery, depending on the technical nature of the request.
    - Once this is complete, the ticket is ready for grooming
    - For a ticket to be ready for grooming, it must meet the following requirements:
        - Includes user story and acceptance criteria
        - Includes some business context on why we&#39;re doing this, why it&#39;s important, etc
        - Developer Tasks - note: this is different from subtasks, which
        - Sprint Candidate tag assigned - ex: S15 Candidate
- **DESIGN**
    - The high level solution it is being discussed.
        - UI
        - Client / Server communication
        - Security implications…etc
    - There may also be times that the Product Manager may have to bring in the Engineering Lead into the discussion to help with discovery, depending on the technical nature of the request.
    - For a ticket to be ready for grooming, it must meet the following requirements:
        - Includes user story and acceptance criteria
        - Includes some business context on why we&#39;re doing this, why it&#39;s important, etc
        - Developer Tasks - note: this is different from subtasks, which
        - Sprint Candidate tag assigned - ex: S15 Candidate
- **GROOMING:**
    - In this step, the developers can, asynchronously, review these tickets and ask clarifying details or ask questions to the Product Manager - usually within the Asana ticket itself
    - The Developer is responsible for:
        - Reviewing the requirements
        - Asking clarifying questions or making comments
        - Writing subtasks (if required)
    - Once this is complete, the ticket is ready to develop
- **READY TO DEVELOP:** 
    - Engineering is confident that the requirements in the Asana ticket are clear, they should move the ticket to READY TO DEVELOP (or the Product Manager can do that)
- **PLANNED:** 
    - The ticket was assigned to some sprint.
- **DONE:** 
    - The ticket is considered done by the PM.
- **ICE BOX:**
    - We&#39;re putting this on hold for now, knowing we may want to tackle it in the future.



### Tech Backlog



This backlog is similar to the regular one but it contains internal technology tasks Like paying tech debt, refactors or library updates.

- Everybody on the team is encouraged to propose tickets to the Tech Lead and he&#39;ll discuss with the Product Manager
- The Engineering Lead is primarily responsible for defining the highest priority items on the backlog, in collaboration with the Product Manager





## Sizing and story points

### Ticket sizing

In order to get good estimations we assign a cost to every ticket. This effort represents the expected time that a task will take to be completed by a mid level developer.

It is important that this scoring system takes into account that a junior developer will make fewer points than a senior developer. Thanks to this, it is not necessary to assign tickets to estimate your effort.

**Story points table**

| Points | Time taken by an developer |
| --- | --- |
| 1 | Less than 1h |
| 2 | - |
| 3 | - |
| 4 | Half a day |
| 5 | - |
| 6 | - |
| 7 | - |
| 8 | One day |

**Sizing limit**

Since software estimates usually have a fairly high margin of error, we try to break down the tickets into small tasks according to the following rules:

- If the ticket is smaller than 8p will be created as usual.
- If the ticket is bigger than 8p but can be done on a single sprint we should create an epic.
    - An epic is just an asana ticket with subtasks.
    - An epic contains all the ticket breakdown.
    - Epics are not added to sprints. **Only their sub-tasks are added to sprints.**
    - Epics are not given points. Its score is the sum of the sub-tasks
- If the ticket cannot be done in a single sprint, we should create an Asana project.

### Sprint sizing



**Weighted estimates**

Since we do not track meetings or routine tasks in Asana and last minute things always appear, we limit the points to a percentage (60%) that each developer can assume per sprint.

| Points per day | Days in a sprint | Points per sprint | Weighted (60%) |
| --- | --- | --- | --- |
| 8p | 10 days | 80p | 48p |

**Point distribution**

We try to distribute sprint points according to the following rules:

- **20% of the time the team will do some trainings.**
- 40% of the time the team will work on new features and bug-fixes.
- 20% of the time the team will work on tech debt tickets.
- 20% of the time is blocked to handle unexpected / unplanned stuff.

### Ticket fields and Labels



A ticket will be tagged as &quot;**[SXX] candidate**&quot; to indicate  which sprint we expect it to be scheduled for. This is a decision primarily driven by the Product Manager, in collaboration with the Tech Lead. This supports asynchronous grooming.

The PM and the Tech lead will review the sprint candidates and verify that they meet all the requirements to be added to a sprint.

The ticket should have the following fields filled out correctly in order to be considered Ready to develop.

- Type: 
- Priority: Helps developers choose the order of tickets in the sprint
- Status: Automatically managed by our automation. 
- Cost: Estimated effort (in story points) —&gt; responsibility of the Tech Lead
- Tags: Once moved to an sprint we should remove the candidate label
- Product:
- Area: 
- Projects: [Optional]
- Dependencies [Optional]
- FF: Indicates if the feature is deployed behind a feature flag or not.

### Ticket status &amp; automation

⚠️ Don&#39;t manually move tickets between sections. Change the status instead.

Among other fields, the tasks have a column called &quot;Status&quot; designed to represent the status of a ticket. This status is cross project Possible values and meaning are listed at[Asana Status Field](https://aptopayments.slab.com/posts/asana-status-field-vl9qp7xm) .

It is important to understand that the **status field does not have a 1: 1 relationship with the column in which the task is represented**. For example, a project like the developer portal can consider that a task is DONE when it is deployed in production because it will be immediately seen by customers. However, projects like the PCI SDK can be deployed in production but are not necessarily DONE until a release is made or a version is published in the apple store or play store.

Because of this we have some ASANA automations which will move the ticket to the right section based on the Status field.



## Sprints

Sprints are intended to be used by engineers to organise the workload. Every 2 weeks the PM will meet the engineers and tickets from the backlog that are ready to be developed will be assigned to the corresponding sprint.

Sprints for Developer Portal are from Monday-following Friday.

![](https://static.slab.com/prod/uploads/fcn5vyb4/posts/images/JS1SnKKs4qwtXAWKJ3TIB-v9.png)

### Sections

Inside a sprint we have the following sections designed to facilitate the work of the engining team.

- **Todo:**  
    - A ticket that is ready to be developed and is expected to be done by the end of the sprint.
    - This is useful for team members to know what they have to work on
- **In progress**
    - This means some developer has started to work on the ticket.
    - This is useful to prevent multiple team members from doing the same task by mistake.
- **In dev review**
    - This means that there is a pull request open for the given task. 
    - This is useful to know which tickets need to be unlocked with a code review.
    - Also includes Product Review on the pull request as Review #1 - only applies to frontend tickets
- **Deployed to staging**
    - This means the pull request was merged to some staging branch.
    - It is useful to know which tickets have to be released to production.
    - Also includes Product Review on staging as Review #2 - only applies to frontend tickets
- **In product review**
    - Tickets here were deployed to staging
    - The PM will do a product review.
        - Discussions will be made in the ticket
            - Why this button is so big?
    - New tickets with issues will be created by PM
    - Tickets will be moved to done once the review step is finished.
- **Done:**  
    - This means the ticket is considered done (usually when deployed to production).
    - It is useful because it allows everyone to get an idea of the progress of the sprint.
    - Also includes Product Review on production as Review #3 - only applies to frontend tickets
    - Note - some tickets may be done, but behind a feature flag. They may not always be visible to the customer.



## Projects

A project is used to represent any kind of work that requires more than one sprint.

![](https://static.slab.com/prod/uploads/fcn5vyb4/posts/images/Lp1nlOlylWRiAzop41ioCT_2.png)

Create new projects using the **DEX_PROJECT_TEMPLATE** to ensure the automation and columns are updated automagically. Projects are automated and every time a task is created inside a project it will be also reflected in the team&#39;s backlog.

![](https://static.slab.com/prod/uploads/fcn5vyb4/posts/images/OgKSODlNMqC7w2ejevvqqUpU.png)

Asana offers a number of interesting features for working with projects.

### Project columns

We follow a D4 methodology where every task can be in one of the following status:

- **Discovery:** The team is researching what needs to be done.
- **Design:** The team is researching how some task will be done.
- **Development:** The engineering team is coding the task.
- **Done:** The team has considered a ticket as completed.

### Project  overview

A section in which you can include the PRD, the files related to Figma&#39;s designs, the participants and their role, the estimated duration of the project.

- Every project roles should include the RACI
- The final PRD should be on SLAB
    - THE PRD SHOULD BE UPDATED WITH THE LATEST AGREEMENTS

![](https://static.slab.com/prod/uploads/fcn5vyb4/posts/images/hxXIHUW_LHmDBhbv-ocl6mvi.png)

### Project  messages

In the messages section you can have asynchronous discussions to make decisions about the epic so that all the information is easily grouped. (Instead of scattering in slack, Google Drive ... etc)

![](https://static.slab.com/prod/uploads/fcn5vyb4/posts/images/EMVYU8KE6Q5MkLPjsUOeSZKh.png)

### Roadmap

By using Asana in this way it is easy to have a roadmap where you can see the sprints, the estimated workload, and the epics that will be worked on during a predetermined period of time.

![](https://static.slab.com/prod/uploads/fcn5vyb4/posts/images/AXtAbECZJcO2umMcg--6XA2T.png)

## Bugs

All reported bugs are taken using our intake form and are automatically listed in the team&#39;s backlog.

The PM will prioritise bugs against other sprints. Bugs will follow the workflow as the backlog and require **PM discovery** before the development team can pull them into a sprint.

## Automation

To avoid having to update the tech status of the tasks manually we use [a cli-tool that can be easily used from a Github Action](https://github.com/AptoPayments/asana-cli) to modify the status of the ticket when certain events occur.

# Git

## Git branches

We follow a policy of one branch per asana ticket. This branch must be named the same as the ID of the associated task. So we can benefit from our automation.

Commits in the branch have no special restrictions.

## Pull requests

Pull requests should be small (To be defined) in order to be easily reviewed. Every pull request should have an approval from a senior and a product approval before being merged in tho the development branch.

Pull requests must be squashed following [the conventional commits convention.](https://www.conventionalcommits.org/en/v1.0.0/)

Every time a PR is opened, an ephemeral backend and frontend environment should created where changes can be tested and in this way the product people can give their feedback.

- Frontend preview is deployed using Netlify.
- Backend preview is not deployed yet TBD.

## Branches

All our repos should have 2 well known branches.

- Main development branch usually called **dev**
- Production branch called usually **prd**

## Releases and environments

In order to ensure high quality while being fast we aim to have an optimal release process.

As soon as the PR is merged to the `dev` branch the changes are deployed on the `staging` environment where anybody can do another round of testing.

Using the Github action &quot;release&quot; though a simple button we can do a release that will include all the changes on staging that are not on production.

![](https://static.slab.com/prod/uploads/fcn5vyb4/posts/images/RqFYzVJ0bq9h4S1akN2hCdbv.png)

**Common rules**

- Every commit to _dev_ is deployed to staging
- Every commit to _prd_ is deployed to production
    1. Developers are not allowed to push to _prd_. They should use the &quot;release&quot; action on Github instead.

# Meetings

To make the most of the team&#39;s productivity, we try to have as few recurring meetings as possible:

**Weekly**

- Frequency: One per week (usually on Wed)
- Duration: ~10m
- Goal:
    - Mostly social / Health check

**Sprint grooming**

- Frequency: One per sprint (mondays)
- Duration: ~30m
- Goal: 
    - Clear out the backlog and detect candidates to the next sprint.

**Sprint planning**

- Frequency: One per sprint (mondays)
- Duration: ~30m
- Goal: 
    - Review and clarify every ticket that will be included into next sprint.

**Retro**

- Frequency: One per sprint (thursday)
- Duration: ~30m
- Goal: Improve team dynamics. 
    - Apto uses [EasyRetro](https://easyretro.io/publicboard/OiVGVUobgCbTC0i3ObaIP0syN7J2/ed9863d4-0d9d-442f-a29a-4a2e462cf14b?sort=votes). 
        - Please use [this link](https://easyretro.io/invite/edbffe5c-b913-4514-998f-667ec3742e97) to join if you don&#39;t have an account.

**Demos**

- Frecuency: Monthly
- We can record our internal meetings with demos and make them public attached to the release notes.
- Goal:
    - Showcase our work

# Feature flags

Feature flags allow us to decouple code deployments from public releases of new features. Thanks to the feature flags we can merge and deploy a feature in production and prevent it from being available to certain groups of users.

See a [list of active feature flags here](https://aptopayments.slab.com/posts/feature-flags-lxf5xi85)

In order to manage feature flags we use a tool called Optimizely.



This tool allows us to define &quot;audiences&quot; / &quot;groups of users&quot; and show / hide certain features to them. At the moment at Apto we have 2 groups:

- Everyone: The default group
- QA: People with the QA mode activated.

This means we can deploy and test a feature to production enabling this feature only for users with the QA role while the users on the group &quot;everyone&quot; won&#39;t notice anything new.

We can also rollout a feature little by little controlling the % of users that are going to see some feature.



Ad-blockers and similar software will block Optimizely and as a result the features might be not available. Because of this, once something is deployed and tested the **final step should be removing the feature flag**



![](https://static.slab.com/prod/uploads/fcn5vyb4/posts/images/h0I63K5wBMJ6UPO7DDhBLr96.png)

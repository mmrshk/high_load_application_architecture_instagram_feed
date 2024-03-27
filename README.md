# Design architecture for an Instagram feed 

## Step 1: Outline use cases and constraints
Who is going to use it?
How are they going to use it?
How many users are there?
What does the system do?
What are the inputs and outputs of the system?
How much data do we expect to handle?
How many requests per second do we expect?
What is the expected read to write ratio?

### Use cases
We'll scope the problem to handle only the following use cases.

- User posts a post into profile
  - Users can upload images and view them  
  - Service pushes posts to followers, sending push notifications
- User views posts feed (activity from people the user is following)
- Service has high availability
- Service should be reliable, ensuring that no photos or videous are lost 
- User can see comments section below the post and also write them
  - Users can like comments

### Out of scope
- Service strips out posts based on users' visibility settings
- Analytics
- Allowing users to comment on a comment
- Allowing users to mute or block someone

## Constraints and assumptions
State assumptions

General
- Posting a post should be fast
  - Fanning out a tweet to all of your followers should be fast, unless you have millions of followers
- Over 500 million daily active users
- 95 million photos per day then per month 2.85 billion photos
  - Let's assume an average fanout of 8 deliveries per post
  - Total Fanout Deliveries per Day = 95 million posts/day * 8 deliveries/post =  760 million deliveries/day
  - Total Fanout Deliveries per Month = 760 million deliveries/day * 30 days/month ≈ 22.8 billion deliveries/month

Timeline

- Viewing the timeline should be fast
- Instagram is more read-heavy than write-heavy
  - Platform should be optimized for fast reeds
- Ingesting posts is write heavy

## Step 2: Create a high-level design

<img width="1108" alt="Screenshot 2024-03-27 at 15 43 33" src="https://github.com/mmrshk/high_load_application_architecture_instagram_feed/assets/31416671/2e932c54-f38e-44cb-93d1-84afd2a2527f">

## Use case: 
### 1. User posts a post into profile

- The LB forwards the request to the Posts and Comments Service directly to Write API server
- Creation of it is handled in SQS 
- The Write API stores the post data in DynamoDB and saves photos/videous on S3
- The Write API contacts the Fan Out Service, which does the following:
  - Queries the User Graph Service to find the user's followers stored in Neo4j, after:
    - It calls User feed generation Service which stores a newly feed in Redis cache
  - Calls Notification Service to send notification to the followers
    - Uses a Queue to asynchronously send out notifications

### 2. User views posts feed

- The LB forwards the request to the User Feed Service directly to Read API
- It gets prepared feed from User Feed Generation Service which is stored in Redis




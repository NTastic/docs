# API Interface Documentation for Front-End Developers

This document provides a comprehensive guide to the GraphQL API endpoints available for your application. It includes details on queries, mutations, types, authentication requirements, and example usages to assist front-end developers in integrating the API effectively.

---

## Table of Contents

1. [Authentication](#authentication)
2. [Scalars and Enums](#scalars-and-enums)
3. [Types](#types)
   - [User](#user)
   - [Tag](#tag)
   - [Question](#question)
   - [Answer](#answer)
   - [Image](#image)
   - [Vote](#vote)
4. [Queries](#queries)
   - [User Queries](#user-queries)
   - [Tag Queries](#tag-queries)
   - [Question Queries](#question-queries)
   - [Answer Queries](#answer-queries)
   - [Image Queries](#image-queries)
5. [Mutations](#mutations)
   - [User Mutations](#user-mutations)
   - [Tag Mutations](#tag-mutations)
   - [Question Mutations](#question-mutations)
   - [Answer Mutations](#answer-mutations)
   - [Vote Mutations](#vote-mutations)
   - [Image Mutations](#image-mutations)
6. [Error Handling](#error-handling)
7. [Example Workflows](#example-workflows)
8. [Additional Considerations](#additional-considerations)

---

## Authentication

- **Method**: JSON Web Tokens (JWT)
- **Header**: `Authorization: Bearer <token>`

**Note**: Authentication is required for mutations that alter user data (e.g., creating questions, answers, voting) and certain queries that retrieve sensitive information.

---

## Scalars and Enums

### Scalars

- `Date`: ISO 8601 formatted date-time string.
- `Upload`: File upload scalar for handling file uploads.

### Enums

- **`SortOrder`**: Defines sorting order.
  - `ASC`: Ascending order.
  - `DESC`: Descending order.

- **`TagMatchType`**: Defines how tags are matched in queries.
  - `ANY`: Match any of the provided tags.
  - `ALL`: Match all of the provided tags.

- **`TagSortField`**: Fields by which tags can be sorted.
  - `name`
  - `questionCount`

- **`TargetType`**: Target type for voting.
  - `Question`
  - `Answer`

- **`VoteType`**: Type of vote.
  - `upvote`
  - `downvote`
  - `cancel` *(Newly added to allow vote cancellation)*

---

## Types

### User

Represents a user in the system.

```graphql
type User {
  id: ID!
  username: String!
  email: String!
  avatar: String
  avatarImageId: ID
  createdAt: Date!
  updatedAt: Date
}
```

### Tag

Represents a tag associated with questions.

```graphql
type Tag {
  id: ID!
  name: String!
  questionCount: Int!
  slug: String!
  description: String
  synonyms: [String]
  parentTag: Tag
  createdAt: Date!
  updatedAt: Date
}
```

### Question

Represents a question posted by a user.

```graphql
type Question {
  id: ID!
  title: String!
  content: String!
  imageIds: [ID!]
  images: [String]
  author: User!
  tags: [Tag!]!
  createdAt: Date!
  updatedAt: Date
  votes: VoteCount!
  answers: [Answer]
}
```

### Answer

Represents an answer to a question.

```graphql
type Answer {
  id: ID!
  content: String!
  imageIds: [ID!]
  images: [String]
  author: User!
  question: Question!
  createdAt: Date!
  updatedAt: Date
  votes: VoteCount!
}
```

### Image

Represents an image uploaded by a user.

```graphql
type Image {
  id: ID!
  filename: String!
  contentType: String!
  length: Int!
  uploadDate: Date
}
```

### Vote

Represents a vote on a question or answer.

```graphql
type Vote {
  id: ID!
  user: User!
  targetId: ID!
  targetType: TargetType!
  voteType: VoteType!
  createdAt: Date!
}
```

### VoteCount

Represents the count of upvotes and downvotes.

```graphql
type VoteCount {
  upvotes: Int!
  downvotes: Int!
}
```

### AuthPayload

Returned upon successful authentication.

```graphql
type AuthPayload {
  token: String!
  user: User!
}
```

### Pagination Types

Used for paginated queries.

**QuestionPagination**

```graphql
type QuestionPagination {
  items: [Question!]!
  totalItems: Int!
  totalPages: Int!
  currentPage: Int!
}
```

**AnswerPagination**

```graphql
type AnswerPagination {
  items: [Answer!]!
  totalItems: Int!
  totalPages: Int!
  currentPage: Int!
}
```

### VoteResponse

Response type for the `vote` mutation.

```graphql
type VoteResponse {
  success: Boolean!
  message: String
  voteCount: VoteCount
}
```

---

## Queries

### User Queries

#### Get User by ID

```graphql
getUser(id: ID!): User
```

**Parameters:**

- `id` (ID!): The ID of the user.

#### Get All Users

```graphql
getUsers: [User]
```

**Note**: Returns a list of all users.

---

### Tag Queries

#### Get Tag by ID

```graphql
getTag(id: ID!): Tag
```

**Parameters:**

- `id` (ID!): The ID of the tag.

#### Get All Tags

```graphql
getTags(
  sort: TagSortInput = { field: name, order: ASC }
): [Tag]
```

**Parameters:**

- `sort` (TagSortInput): Optional sorting input.
  - `field` (TagSortField!): Field to sort by (`name`, `questionCount`).
  - `order` (SortOrder!): Sort order (`ASC`, `DESC`).

#### Search Tags

```graphql
searchTags(keyword: String!): [Tag]
```

**Parameters:**

- `keyword` (String!): Keyword to search in tag names and synonyms.

---

### Question Queries

#### Get Question by ID

```graphql
getQuestion(id: ID!): Question
```

**Parameters:**

- `id` (ID!): The ID of the question.

#### Get Questions with Filters and Pagination

```graphql
getQuestions(
  tagIds: [ID!]
  tagMatch: TagMatchType = ANY
  userId: ID
  page: Int = 1
  limit: Int = 10
  sortOrder: SortOrder = DESC
): QuestionPagination!
```

**Parameters:**

- `tagIds` ([ID!]): List of tag IDs to filter questions.
- `tagMatch` (TagMatchType): `ANY` or `ALL`.
- `userId` (ID): Filter questions by author.
- `page` (Int): Page number for pagination.
- `limit` (Int): Number of items per page.
- `sortOrder` (SortOrder): `ASC` or `DESC` for sorting by `createdAt`.

---

### Answer Queries

#### Get Answer by ID

```graphql
getAnswer(id: ID!): Answer
```

**Parameters:**

- `id` (ID!): The ID of the answer.

#### Get Answers with Filters and Pagination

```graphql
getAnswers(
  questionId: ID
  userId: ID
  page: Int = 1
  limit: Int = 10
  sortOrder: SortOrder = ASC
): AnswerPagination!
```

**Parameters:**

- `questionId` (ID): Filter answers by question.
- `userId` (ID): Filter answers by author.
- `page` (Int): Page number for pagination.
- `limit` (Int): Number of items per page.
- `sortOrder` (SortOrder): `ASC` or `DESC` for sorting by `createdAt`.

**Note**: At least one of `questionId` or `userId` must be provided.

---

### Image Queries

#### Get Image by ID

```graphql
getImage(id: ID!): Image
```

**Parameters:**

- `id` (ID!): The ID of the image.

#### Get User's Images

```graphql
getUserImages: [Image!]!
```

**Authentication Required**

**Note**: Returns a list of images uploaded by the authenticated user.

---

## Mutations

### User Mutations

#### Register User

```graphql
register(username: String!, email: String!, password: String!): AuthPayload
```

**Parameters:**

- `username` (String!): Desired username.
- `email` (String!): User's email address.
- `password` (String!): Password.

**Returns:**

- `AuthPayload`: Contains `token` and `user` object.

#### Login User

```graphql
login(email: String!, password: String!): AuthPayload
```

**Parameters:**

- `email` (String!): User's email address.
- `password` (String!): Password.

**Returns:**

- `AuthPayload`: Contains `token` and `user` object.

#### Update User

```graphql
updateUser(username: String, avatarImageId: ID): User
```

**Authentication Required**

**Parameters:**

- `username` (String): New username.
- `avatarImageId` (ID): ID of the new avatar image.

---

### Tag Mutations

#### Create Tag

```graphql
createTag(
  name: String!,
  description: String,
  synonyms: [String],
  parentTagId: ID
): Tag
```

**Authentication Required**

**Parameters:**

- `name` (String!): Name of the tag.
- `description` (String): Description of the tag.
- `synonyms` ([String]): List of synonyms.
- `parentTagId` (ID): ID of the parent tag.

#### Update Tag

```graphql
updateTag(
  id: ID!,
  name: String,
  description: String,
  synonyms: [String],
  parentTagId: ID
): Tag
```

**Authentication Required**

**Parameters:**

- `id` (ID!): ID of the tag to update.
- `name` (String): New name.
- `description` (String): New description.
- `synonyms` ([String]): New synonyms.
- `parentTagId` (ID): New parent tag ID.

#### Merge Tags

```graphql
mergeTags(
  sourceTagIds: [ID!]!,
  targetTagId: ID!
): MergeTagsResponse
```

**Authentication Required**

**Parameters:**

- `sourceTagIds` ([ID!]!): IDs of tags to merge.
- `targetTagId` (ID!): ID of the target tag.

**Returns:**

- `MergeTagsResponse`: Contains `success`, `message`, and `mergedTag`.

---

### Question Mutations

#### Create Question

```graphql
createQuestion(
  title: String!,
  content: String!,
  tagIds: [ID!]!,
  imageIds: [ID!]
): Question
```

**Authentication Required**

**Parameters:**

- `title` (String!): Title of the question.
- `content` (String!): Content of the question.
- `tagIds` ([ID!]!): IDs of associated tags.
- `imageIds` ([ID!]): IDs of associated images.

#### Update Question

```graphql
updateQuestion(
  id: ID!,
  title: String,
  content: String,
  tagIds: [ID!],
  imageIds: [ID!]
): Question
```

**Authentication Required**

**Parameters:**

- `id` (ID!): ID of the question to update.
- `title` (String): New title.
- `content` (String): New content.
- `tagIds` ([ID!]): New tag IDs.
- `imageIds` ([ID!]): New image IDs.

#### Delete Question

```graphql
deleteQuestion(id: ID!): Boolean!
```

**Authentication Required**

**Parameters:**

- `id` (ID!): ID of the question to delete.

**Returns:**

- `Boolean`: `true` if deletion was successful.

---

### Answer Mutations

#### Create Answer

```graphql
createAnswer(
  questionId: ID!,
  content: String!,
  imageIds: [ID!]
): Answer
```

**Authentication Required**

**Parameters:**

- `questionId` (ID!): ID of the question to answer.
- `content` (String!): Content of the answer.
- `imageIds` ([ID!]): IDs of associated images.

#### Update Answer

```graphql
updateAnswer(
  id: ID!,
  content: String,
  imageIds: [ID!]
): Answer
```

**Authentication Required**

**Parameters:**

- `id` (ID!): ID of the answer to update.
- `content` (String): New content.
- `imageIds` ([ID!]): New image IDs.

#### Delete Answer

```graphql
deleteAnswer(id: ID!): Boolean!
```

**Authentication Required**

**Parameters:**

- `id` (ID!): ID of the answer to delete.

**Returns:**

- `Boolean`: `true` if deletion was successful.

---

### Vote Mutations

#### Vote on Question or Answer

```graphql
vote(
  targetId: ID!,
  targetType: TargetType!,
  voteType: VoteType!
): VoteResponse
```

**Authentication Required**

**Parameters:**

- `targetId` (ID!): ID of the question or answer.
- `targetType` (TargetType!): `Question` or `Answer`.
- `voteType` (VoteType!): `upvote`, `downvote`, or `cancel`.

**Returns:**

- `VoteResponse`: Contains `success`, `message`, and updated `voteCount`.

---

### Image Mutations

#### Upload Image

```graphql
uploadImage(file: Upload!): Image!
```

**Authentication Required**

**Parameters:**

- `file` (Upload!): File to upload.

**Returns:**

- `Image`: Uploaded image information.

#### Delete Image

```graphql
deleteImage(imageId: ID!): Boolean!
```

**Authentication Required**

**Parameters:**

- `imageId` (ID!): ID of the image to delete.

**Returns:**

- `Boolean`: `true` if deletion was successful.

---

## Error Handling

- **Authentication Errors**: If authentication fails or the user is not authorized, an error with a message like `"Authentication required"` or `"You are not authorized to perform this action"` will be returned.
- **Validation Errors**: If input validation fails, errors with messages indicating the invalid fields will be returned.
- **Not Found Errors**: If a requested resource is not found, an error with a message like `"Question not found"` or `"User not found"` will be returned.
- **General Errors**: For other failures, a generic error message will be returned. Detailed error information should be logged on the server for debugging.

---

## Example Workflows

### 1. User Registration and Login

**Register a User**

```graphql
mutation {
  register(username: "john_doe", email: "john@example.com", password: "password123") {
    token
    user {
      id
      username
      email
    }
  }
}
```

**Login a User**

```graphql
mutation {
  login(email: "john@example.com", password: "password123") {
    token
    user {
      id
      username
      email
    }
  }
}
```

**Note**: Store the `token` from the response and include it in the `Authorization` header for authenticated requests.

---

### 2. Creating a Question

**Upload Images**

```graphql
mutation($file: Upload!) {
  uploadImage(file: $file) {
    id
    filename
  }
}
```

**Variables**:

```json
{
  "file": null
}
```

**Note**: When using `graphql-upload`, the file should be provided through the form-data interface.

**Create a Question**

```graphql
mutation {
  createQuestion(
    title: "How to implement authentication in GraphQL?",
    content: "I'm looking for best practices...",
    tagIds: ["TAG_ID_1", "TAG_ID_2"],
    imageIds: ["IMAGE_ID_1"]
  ) {
    id
    title
    content
    images
    tags {
      name
    }
    author {
      username
    }
  }
}
```

---

### 3. Answering a Question

**Create an Answer**

```graphql
mutation {
  createAnswer(
    questionId: "QUESTION_ID",
    content: "You can use JWT tokens and middleware...",
    imageIds: ["IMAGE_ID_2"]
  ) {
    id
    content
    author {
      username
    }
  }
}
```

---

### 4. Voting on a Question

**Upvote a Question**

```graphql
mutation {
  vote(
    targetId: "QUESTION_ID",
    targetType: Question,
    voteType: upvote
  ) {
    success
    message
    voteCount {
      upvotes
      downvotes
    }
  }
}
```

**Cancel a Vote**

```graphql
mutation {
  vote(
    targetId: "QUESTION_ID",
    targetType: Question,
    voteType: cancel
  ) {
    success
    message
    voteCount {
      upvotes
      downvotes
    }
  }
}
```

**Change Vote from Upvote to Downvote**

```graphql
mutation {
  vote(
    targetId: "QUESTION_ID",
    targetType: Question,
    voteType: downvote
  ) {
    success
    message
    voteCount {
      upvotes
      downvotes
    }
  }
}
```

---

### 5. Fetching Questions with Filters

**Get Questions with Specific Tags**

```graphql
query {
  getQuestions(
    tagIds: ["TAG_ID_1", "TAG_ID_2"],
    tagMatch: ANY,
    page: 1,
    limit: 10,
    sortOrder: DESC
  ) {
    items {
      id
      title
      content
      tags {
        name
      }
      author {
        username
      }
      createdAt
    }
    totalItems
    totalPages
    currentPage
  }
}
```

---

## Additional Considerations

### Authentication Tokens

- **Expiration**: Tokens expire after 1 day (`expiresIn: '1d'`).
- **Refresh Tokens**: Not implemented. Users need to log in again after expiration.

### Image Handling

- **Serving Images**: Images are served via a GET endpoint at `/images/:id`.
- **Image URLs**: Use the `images` field in `Question` and `Answer` types, which provide the full URL.

### Data Consistency

- **Transactions**: Deletion operations are performed without transactions. Be cautious of potential data inconsistencies in concurrent environments.

### Error Messages

- **Sensitive Information**: Error messages avoid exposing sensitive information. For example, login errors do not specify whether the email or password was incorrect.

### Rate Limiting

- **Limits**: The server enforces rate limiting to prevent abuse. Adjust client requests accordingly.

### CORS Configuration

- **Allowed Origins**: The server allows origins specified in the `ALLOWED_ORIGINS` environment variable.

---

## Contact and Support

For questions or issues related to the API, please contact the backend development team or refer to the project's documentation repository.

---

**Happy Coding!**

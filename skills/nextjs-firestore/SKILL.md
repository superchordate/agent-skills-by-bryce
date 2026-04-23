---
name: nextjs-firestore
description: Guide for working with Firebase Firestore in Next.js applications. Use when reading/writing Firestore documents, setting up indexes, creating human-readable IDs, or handling client vs server-side Firestore operations. Covers required document fields, index management, and code examples for both client and server contexts.
---

# Next.js Firestore Integration Guide

## Quick Reference

**Core Concepts:**
- [Client vs Server](#client-vs-server) - Different imports and patterns for each context
- [General Requirements](#general-requirements) - ID format, indexes, required fields
- [Required Document Fields](#required-document-fields) - createdAt, lastModified, createdBy, lastModifiedBy

**Common Tasks:**
- Reading documents → [Client examples](#client-operations) | [Server examples](#server-operations)
- Creating documents → [Client](#creating-documents-client) | [Server](#creating-documents-server)
- Querying with indexes → [Index setup](#firestore-indexes)
- Creating human-readable IDs → [ID format guide](#human-readable-ids)

**Key Rules:**
- Use client imports (`firebase/firestore`) in Client Components
- Use server imports (`@/lib/firebase/serverApp`) in Server Components/API routes
- Always include required fields: createdAt, lastModified, createdBy, lastModifiedBy
- Deploy indexes after adding queries: `firebase deploy --only firestore:indexes`

## General Requirements

### Human-Readable IDs

Firestore document IDs should always be **human-readable** and follow this pattern:

**Format:** `{index-information}-{random-hash}`

**Examples:**
- `user-john-doe-a3f9`
- `order-20260409-b7k2`
- `token-alice-20260409-x4m8`

**Guidelines:**
- Combine index information (user ID, date, etc.) with a random 4-6 digit alphanumeric hash
- Use hashes to prevent ID collisions when same index fields might be reused
- Keep IDs concise but meaningful for debugging

### Firestore Indexes

**When to add indexes:** Whenever you create a query with multiple conditions or ordering.

**Process:**
1. Add the index definition to `firestore.indexes.json`
2. Deploy: `firebase deploy --only firestore:indexes`
3. Wait for index to build (can take a few minutes)

**Example index definition:**
```json
{
  "indexes": [
    {
      "collectionGroup": "users",
      "queryScope": "COLLECTION",
      "fields": [
        { "fieldPath": "active", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ]
}
```

### Required Document Fields

**Every Firestore document must include these four fields:**

| Field | Type | Description | Reference Target |
|-------|------|-------------|------------------|
| `createdAt` | Timestamp | When record was created | N/A |
| `lastModified` | Timestamp | When record was last modified | N/A |
| `createdBy` | Reference | User who created the record | `users/{username}` |
| `lastModifiedBy` | Reference | User who last modified the record | `users/{username}` |

**Important:** User references should link to the **username**, not the Firebase auth UID.

## Client vs Server

Firestore operations differ significantly between **client-side** (Client Components) and **server-side** (Server Components, API routes) contexts.

### Client Operations

**Import pattern:**
```js
import {
  collection,
  getDocs,
  doc,
  getDoc,
  addDoc,
  updateDoc,
  deleteDoc,
  query,
  where,
  orderBy,
  serverTimestamp,
} from "firebase/firestore";
import { db } from "@/lib/firebase/firebaseConfig";
```

#### Reading Documents (Client)

**Single document:**
```js
// Get a user by ID
const docRef = doc(db, "users", "userId");
const docSnap = await getDoc(docRef);

if (docSnap.exists()) {
  const userData = docSnap.data();
  console.log("User:", userData);
} else {
  console.log("No such document!");
}
```

**Multiple documents:**
```js
// Query active users
const q = query(
  collection(db, "users"),
  where("active", "==", true),
  orderBy("createdAt", "desc")
);
const querySnapshot = await getDocs(q);

const users = querySnapshot.docs.map(doc => ({
  id: doc.id,
  ...doc.data()
}));
```

#### Creating Documents (Client)

**With auto-generated ID:**
```js
import { collection, addDoc, serverTimestamp } from "firebase/firestore";

const docRef = await addDoc(collection(db, "users"), {
  name: "John Doe",
  email: "john@example.com",
  active: true,
  createdAt: serverTimestamp(),
  lastModified: serverTimestamp(),
  createdBy: doc(db, "users", currentUsername),
  lastModifiedBy: doc(db, "users", currentUsername),
});

console.log("Document created with ID:", docRef.id);
```

**With custom ID:**
```js
import { doc, setDoc, serverTimestamp } from "firebase/firestore";

// Create human-readable ID
const randomHash = Math.random().toString(36).substring(2, 6);
const customId = `user-${username}-${randomHash}`;

await setDoc(doc(db, "users", customId), {
  name: "John Doe",
  email: "john@example.com",
  active: true,
  createdAt: serverTimestamp(),
  lastModified: serverTimestamp(),
  createdBy: doc(db, "users", currentUsername),
  lastModifiedBy: doc(db, "users", currentUsername),
});
```

#### Updating Documents (Client)

```js
import { doc, updateDoc, serverTimestamp } from "firebase/firestore";

const docRef = doc(db, "users", "userId");
await updateDoc(docRef, {
  active: false,
  lastModified: serverTimestamp(),
  lastModifiedBy: doc(db, "users", currentUsername),
});
```

#### Deleting Documents (Client)

```js
import { doc, deleteDoc } from "firebase/firestore";

const docRef = doc(db, "users", "userId");
await deleteDoc(docRef);
```

### Server Operations

**Import pattern:**
```js
import { db } from "@/lib/firebase/serverApp";
import { FieldValue } from "firebase-admin/firestore";
```

#### Reading Documents (Server)

**Single document:**
```js
// Get a user by ID
const docRef = db.collection("users").doc("userId");
const docSnap = await docRef.get();

if (docSnap.exists) {
  const userData = docSnap.data();
  console.log("User:", userData);
} else {
  console.log("No such document!");
}
```

**Multiple documents:**
```js
// Query active users
const querySnapshot = await db
  .collection("users")
  .where("active", "==", true)
  .orderBy("createdAt", "desc")
  .get();

const users = querySnapshot.docs.map(doc => ({
  id: doc.id,
  ...doc.data()
}));
```

#### Creating Documents (Server)

**With custom ID:**
```js
import { FieldValue } from "firebase-admin/firestore";

// Create human-readable ID
const today = new Date().toISOString().split('T')[0].replace(/-/g, '');
const randomHash = Math.random().toString(36).substring(2, 8);
const id = `token-${userId}-${today}-${randomHash}`;

const data = {
  userId: userId,
  token: "abc123",
  active: true,
  createdAt: FieldValue.serverTimestamp(),
  lastModified: FieldValue.serverTimestamp(),
  createdBy: db.collection("users").doc(currentUsername),
  lastModifiedBy: db.collection("users").doc(currentUsername),
};

await db.collection("tokens").doc(id).set(data);
```

**With auto-generated ID:**
```js
const docRef = await db.collection("users").add({
  name: "John Doe",
  email: "john@example.com",
  active: true,
  createdAt: FieldValue.serverTimestamp(),
  lastModified: FieldValue.serverTimestamp(),
  createdBy: db.collection("users").doc(currentUsername),
  lastModifiedBy: db.collection("users").doc(currentUsername),
});

console.log("Document created with ID:", docRef.id);
```

#### Updating Documents (Server)

```js
import { FieldValue } from "firebase-admin/firestore";

const docRef = db.collection("users").doc("userId");
await docRef.update({
  active: false,
  lastModified: FieldValue.serverTimestamp(),
  lastModifiedBy: db.collection("users").doc(currentUsername),
});
```

#### Deleting Documents (Server)

```js
const docRef = db.collection("users").doc("userId");
await docRef.delete();
```

## Best Practices

**✅ DO:**
- Always include all four required fields (createdAt, lastModified, createdBy, lastModifiedBy)
- Use human-readable IDs with random hashes to prevent collisions
- Deploy indexes immediately after adding new queries
- Use server-side operations in API routes and Server Components
- Use client-side operations in Client Components only
- Reference users by username, not Firebase auth UID

**❌ DON'T:**
- Skip required fields to "save time" - they're essential for auditing
- Use sequential IDs without randomization (risk of collisions)
- Forget to deploy indexes (queries will fail in production)
- Mix client and server import patterns
- Reference users by Firebase auth UID in document metadata

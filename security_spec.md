# Firebase Security Specification - Grameen-Light

## 1. Data Invariants
- A Complaint must reference a valid Pole ID.
- Status transitions must follow logical flow (Pending -> Assigned -> Fixed).
- Only authenticated users can file reports (for now, assuming anonymous or Google).
- Panchayat official notes (Dairy entries) must have an author.

## 2. The Dirty Dozen Payloads (Rejections)
1. **Identity Spoofing**: Attempt to file a complaint as another user.
2. **State Shortcutting**: Update a 'Pending' complaint directly to 'Fixed' without 'Assigned' (if enforced, though here we might allow skip for simplicity in some cases).
3. **Shadow Update**: Add a field `isVerified: true` to a Pole document.
4. **ID Poisoning**: Use a 2KB string as a Pole ID.
5. **PII Leak**: Read private notes if any (not implemented yet).
6. **Denial of Wallet**: Update 'history' array with 10,000 elements.
7. **Type Mismatch**: Send `lat: "north"` instead of a number.
8. **Immutability Breach**: Change `poleId` on an existing complaint.
9. **Relational Orphan**: Create a complaint for a pole that doesn't exist in the 'poles' collection.
10. **Timestamp Fraud**: Set `timestamp` to a future date from the client.
11. **Blanket Read Attack**: Query `complaints` without any filters as an unauthenticated user.
12. **Malicious Attachment**: Set `attachments` as a 1MB string instead of a list.

## 3. Enforcement Strategy
- `isValidPole()` and `isValidComplaint()` helpers.
- `affectedKeys().hasOnly()` for specific state updates.
- `exists()` checks for relational integrity.

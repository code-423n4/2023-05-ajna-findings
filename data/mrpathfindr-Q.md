

Remove Unnecessary Comment
---

Link to issue: https://github.com/code-423n4/2023-05-ajna/blob/276942bc2f97488d07b887c8edceaaab7a5c3964/ajna-grants/src/grants/base/Funding.sol#L127

Recommendation: Remove comment.

Add error message to // already joined in Comptroller.sol 
---

POC:

```
if (marketToJoin.accountMembership[borrower]) {
            // already joined
            return; //@audit this will return with no error message?!?! Sinc the user has already joined should it not revert with error message?
        }
```

Recommendation: Replace return with, Revert this code block and return error message 
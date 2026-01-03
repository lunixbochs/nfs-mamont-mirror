# Retransmission (DRC) TODO

## 1) Cache reply bytes in TransactionTracker
- Change TransactionState to store response bytes for completed transactions.
  - Example: Completed { at: SystemTime, response: Arc<Vec<u8>> }
- Keep a bounded retention policy (existing TTL + optional max entries).

## 2) Expose richer lookup API
- Replace `is_retransmission()` with a method that distinguishes states:
  - New
  - InProgress
  - Completed(response)
- Suggested signature:
  - `check(xid, client_addr) -> TransactionStatus`

## 3) Persist response bytes at the right point
- Store the serialized reply bytes after the handler writes to the response buffer.
- Avoid the current race where `mark_processed()` is called before response is cached.
- This likely means:
  - Remove `mark_processed()` from `handle_rpc()`
  - Call something like `record_response(xid, client_addr, response_bytes)`
    in the command-queue path after the response buffer is finalized.

## 4) Reply on retransmit
- In `handle_rpc` (or the command dispatcher):
  - If status is Completed(response), write cached bytes to output and return `Ok(true)`.
  - If InProgress, ignore or delay duplicate (no re-execution).

## 5) Tests
- Add a test that sends the same XID twice and asserts the same reply bytes.
- Add a test that retransmits while the original call is in-flight (if feasible).

---

TODO: NFSv3 tests from https://github.com/phdeniel/cthon04
TODO: ganesha's mountpoint tests via a mounted share `src/scripts/test_through_mountpoint/`
TODO: lima vm for testing
TODO: require low port bindings
TODO: override `readdir_index` in backends to avoid rescanning from the start

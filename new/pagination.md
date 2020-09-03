# Pagination

Pagination needs to be handled by client application. Server does not keep the resultset fetching status.

The typical scenario:

1. Client call `!search` endpoint. It can specify some filter conditions and sort options.
2. Server returns up to `maxItems` IDs.
3. Client sends part of these IDs to `!batchGet` endpoint.
4. Server returns detail objects for specified IDs.
5. Client get another part of IDs returned by `!search` endpoint (e.g. when the user scrolls down in the displayed list of records) and sends them to the `!batchGet` operation. This step can be repeated many times.

When there is more then `maxItems` records in the DB which fullfils specified conditions then `params.maxItemsApplied = true` is returned by `!search` endpoint. In this case returned IDs are ordered list of randomly choosen subset of all appropriate records. The client needs to repeat the `!search` operation with more restrictive filter conditions in such case.


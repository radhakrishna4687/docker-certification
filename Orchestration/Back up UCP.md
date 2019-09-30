# Back up UCP
- UCP backups no longer require pausing the reconciler and deleting UCP containers, and backing up a UCP manager does not disrupt the manager’s activities.
- Because UCP stores the same data on all manager nodes, you only need to back up a single UCP manager node.
- User resources, such as services, containers, and stacks are not affected by this operation and continue operating as expected.
# UCP backup steps
There are several options for creating a UCP backup:
- CLI
- UI
- API

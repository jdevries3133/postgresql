# Original Email

Hi,

We are upgrading from Postgres 11 to 13. During upgrade we found that the
physical slots on the old cluster are not copied to the new cluster.

This information is not mentioned in the documentation -
https://www.postgresql.org/docs/13/pgupgrade.html

Just thought it would be good to have this detail

Thanks and Regards,
Nikhil

# Current Progress

## GitLab Migration Example

Some good info about what goes into a replicated upgrade, from when GitLab
upgraded from v9 to v11

https://gitlab.com/gitlab-com/gl-infra/reliability/-/issues/9097

## Relevant PostgreSQL Docs

- ["Logical Decoding Concepts"](https://www.postgresql.org/docs/9.4/logicaldecoding-explanation.html#AEN67208)
- ["pg_replication_slots"](https://www.postgresql.org/docs/9.4/catalog-pg-replication-slots.html)
- ["Log-Shipping Standby Servers"](https://www.postgresql.org/docs/9.4/warm-standby.html#STREAMING-REPLICATION-SLOTS)

Basically, replication slots store changes that still need to be propagated to
replicas, and is presumably purged after the receipt of those changes is
communicated by replicas.

It seems very strange to me that this would be overlooked by the `pg_upgrade`
utility. My suspicion is that there is support for this in the source, I just
need to find it. If there is no mention of this in the source for `pg_upgrade`,
that is indeed very strange, and the behavior should be documented. Maybe there
is even interest in backporting a fix because it doesn't seem like this should
be a thing that happens.

## Docs Changes 11 to 13 and 13 to HEAD

I reviewed the diffs from v11 to v13, and then from v13 to HEAD of the
`pgupgrade.sgml` file. No changes specifically related to physical replication
slots were noted.

# Close Out

I replied to the emailer, suggesting that following step 9 of pgupgrade might
mitigate the issue.

> Hi Nikhil,
>
> From the pgupgrade docs:
>
> > 9. Prepare for standby server upgrades
> >
> > If you are upgrading standby servers using methods outlined in section
> > Step 11, verify that the old standby servers are caught up by running
> > pg_controldata against the old primary and standby clusters. Verify
> > that the “Latest checkpoint location” values match in all clusters.
> > (There will be a mismatch if old standby servers were shut down before
> > the old primary or if the old standby servers are still running.)
> > Also, make sure wal_level is not set to minimal in the postgresql.conf
> > file on the new primary cluster.
>
> (source: https://www.postgresql.org/docs/devel/pgupgrade.html)
>
> I'm a new contributor so please forgive me if I'm on the wrong track,
> but if you follow this step, won't you also be ensuring that replication
> slots do not need to be migrated, since you've just ensured that standby
> clusters are in sync with the primary cluster? Please let me know if I'm
> missing anything!
>
> Thank You,
> Jack DeVries

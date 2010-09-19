# Attention: DEFECT

There is a known defect in this patch currently. The entity that is coalesced is still marked
as dirty in the session post flush. This is being worked on now.

# Introduction

Thank you to [Julian Klein](http://twitter.com/juliank) for his help on making this patch.

This is a patched version of Hibernate 3.5.6-Final. The patch is to enhance the
flushing behavior of Hibernate and cause it to "coalesce" the UPDATE into the 
INSERT even when there were dirty changes made to the entity post merge/persist.

What this boils down to is if you merge/persist an entity into a Hibernate session,
and then make changes to that managed object, Hibernate ends up executing two 
DML operations instead of one at flush time - an INSERT and then an UPDATE. The INSERT
represents the state of the entity at the moment it was merged or persisted, and the
UDPATE syncs up any other changes that were made to it after it became managed.

This patch will cause Hibernate to "coalesce" the UPDATE into the INSERT, and results
in a single INSERT DML operation at flush time.

For more links on this issue, and why people care, see [these links.](http://delicious.com/sfraser/coalesce)

The files of interest modified in this patch are:

[EntityInsertAction](http://github.com/sfraser/Hibernate-Core-3.5.6-Final-patched-with--True-Coalesce--enhancement/commit/33290bfc0b8ba8aef443eb029c8ed2aa728743c9#diff-3)

[ActionQueue](http://github.com/sfraser/Hibernate-Core-3.5.6-Final-patched-with--True-Coalesce--enhancement/commit/33290bfc0b8ba8aef443eb029c8ed2aa728743c9#diff-4)

[DefaultFlushEntityEventListener](http://github.com/sfraser/Hibernate-Core-3.5.6-Final-patched-with--True-Coalesce--enhancement/commit/33290bfc0b8ba8aef443eb029c8ed2aa728743c9#diff-5)


To confirm this patch works you would want to monitor the SQL executed at flush time.

An example of a unit test to confirm this patch works as expected might look like this:

`
    @Autowired
    private ReimburseSystemDao reimburseSystemDao;

    @PersistenceContext
    EntityManager em;

    /**
     * This function tests that only one insert is created
     * over the span of the transaction when working with
     * an object that is attached and subsequently altered.
     * The test also ensures no updates are generated.
     */
    @Test
    @Transactional
    @Rollback(true)
    public void testToEnsureOneInsertIsExecuted() {

        //create the object
        Description desc = new Description();
        desc.setBrief("TEST");
        desc.setExtended("TEST DSL");
        ReimburseSystem reimburseSystem = new ReimburseSystem();
        reimburseSystem.setDescription(desc);

        //attach the object to the session
        reimburseSystem = reimburseSystemDao.merge(reimburseSystem);

        //make sure the id was generated
        assertNotNull(reimburseSystem.getId());
        if(!(reimburseSystem.getId() > 0)) {
            fail("sequence should have produced an id");
        }

        //alter the object, which traditionally causes
        //hibernate to create an UPDATE event in Oracle
        reimburseSystem.setReimburseSystemCode("A100");

        //enable and get the stats so we can ensure that
        //we get the expected behavior
        Statistics stats = ((Session)em.getDelegate()).getSessionFactory().getStatistics();
        stats.setStatisticsEnabled(true);

        //get the current stats for updates and inserts
        final long preFlushUpdateCount = stats.getEntityUpdateCount();
        final long preFlushInsertCount = stats.getEntityInsertCount();

        //make sure no inserts or updates have occurred
        assertEquals(preFlushUpdateCount, 0);
        assertEquals(preFlushInsertCount, 0);

        //flush the session to ensure all DML is executed
        em.flush();

        //get the post flush stats to makes sure everything went according to plan
        final long postFlushUpdateCount = stats.getEntityUpdateCount();
        final long postFlushInsertCount = stats.getEntityInsertCount();

        //make sure the stats for updates are equals to the before
        //state (in other words no changes should have happened)
        assertEquals(postFlushUpdateCount, preFlushUpdateCount);

        //make sure no updates have occurred
        assertEquals(postFlushUpdateCount, 0);

        //make sure we only got one insert
        assertEquals(postFlushInsertCount, 1);
    }
`
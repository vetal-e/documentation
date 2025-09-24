<a id="dev-entities-entity-manager"></a>

# Entity Manager

Class <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EntityBundle/ORM/OroEntityManager.php" target="_blank">OroEntityManager</a> is used to extend some native Doctrine Entity Manager functionality. If any other modifications are required, your class should extend OroEntityManager instead of the Doctrine Entity Manager.

**Additional ORM Lifecycle Events**

In addition to standard <a href="https://www.doctrine-project.org/projects/doctrine-orm/en/2.8/reference/events.html#lifecycle-events" target="_blank">Doctrine ORM Lifecycle Events</a>, the OroEntityManager triggers new events:

- *preClose* - The preClose event occurs when the EntityManager#close() operation is invoked, before EntityManager#clear() is invoked.

<!-- Frontend -->

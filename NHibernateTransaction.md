# Always use transactions when dealing with NHibernate

Today I learned that NHibernate will only save all the things in the context of a transaction.

We were dealing with a more complicated mapping where our object had a reference to another kind of object and a list of strings to persist along with basic properties.  Something like this:

```C#
namespace Steve.Reports
{
    public class Export
    {
        public virtual int Id { get; set; }
        public virtual DateTime StartDateTime { get; set; }
        public virtual DateTime EndDateTime { get; set; }
        public virtual ICollection<string> Locations { get; set; }
        public virtual Layout Layout { get; set; }
   }
}
```

Since this involved some slightly more involved NHibernate ninjitsu than we were comfortable with we wanted to try and get it working with some fast feedback.  So we built an automated test to throw away later.

```C#
        [Test]
        public void TestCreate()
        {
            var layout = new Layout { Name = "Garpster" };
            var export = new Export {
                StartDateTime = DateTime.Now,
                EndDateTime = DateTime.Now,
                Locations = new List<string> { "1", "2" },
                Layout = layout
            };

            session.Save(export);
            session.Clear();

            var fetchedExport = session.Get<Export>(export.Id);
            Assert.AreEqual(export, fetchedExport);
        }
```

The idea here is to create the object, save it to the database and then pull it back and see if it's the same.

The surprising thing in retrospect is that the layout persisted correctly, the export persisted correctly, but the locations did not.

We spent hours questioning our DB schema, NHibernate mappings, and sanity until Huey saved the day.

It seems NHibernate doesn't flush everything to the database until being told to.  So you can either add a call to session.Flush() or do the operation like we would in production: wrap it in a transaction.

```C#
            //object building up here
            using (var tx = session.BeginTransaction())
            {
                session.Save(export);
                tx.Commit();
            }
            //assertions down here
```

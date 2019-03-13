## Examples

### Serialization
#### Graph
Serialize a graph into a json string.
```
Graph graph = new MemGraph();
---
String json = GraphSerializer.toJson(graph);
```
Deserialize a json string to a graph.
```
Graph graph = GraphSerializer.fromJson(new MemGraph(), json);
```
#### Prohibitions
Serialize a ProhibitionDAO into a json string.
```
ProhibitionsDAO dao = new MemProhibitionsDAO();
---
String json = ProhibitionsSerializer.toJson(dao);
```
Deserialize a json string to a ProhibitionsDAO.
```   
ProhibitionsDAO deDao = ProhibitionsSerializer.fromJson(new MemProhibitionsDAO(), json);
```


### Bank Teller

#### Graph configuration summary

- Users: u1, u2
- An object o1
- Two policy classes: RBAC and Branches
    - RBAC
        - o1 is assigned to accounts
        - u1 is a Teller that has read and write permissions on accounts
        - u2 is an Auditor that has read permissions on accounts
    - Branches
        - u1 and u2 are both assigned to the Branch 1 user attribute
        - o1 is assigned to the Branch 1 object attribute
        - the Branch 1 user attribute has read and write permissions on the Branch 1 object attribute

#### Access control state

- u1 can read and write o1
- u2 can read o1
---

#### Code Wakthrough
```
// 1. Create a new Graph instance.  For this example, we'll use the `MemGraph` which is an in memory implementation of the Graph interface.
Graph graph = new MemGraph();

// 2. Create the user nodes `u1` and `u2`.
long user1ID = graph.createNode(rand.nextLong(), "u1", U, null));
long user2ID = graph.createNode(rand.nextLong(), "u2", U, null));


// 3. Create the object, `o1` that will be the target of the access queries.
long objectID = graph.createNode(rand.nextLong(), "o1", O, null));


// 4. Create the `RBAC` policy class node.
long rbacID = graph.createNode(rand.nextLong(), "RBAC", PC, null));


// 5. Create an object attribute for the `Accounts`.
long accountsID = graph.createNode(rand.nextLong(), "Accounts", OA, null));


// 6. Create the `Teller` and `Auditor` user attributes.
long tellerID = graph.createNode(rand.nextLong(), "Teller", UA, null));
long auditorID = graph.createNode(rand.nextLong(), "Auditor", UA, null));


// 7. Assign the `Accounts` object attribute to the `RBAC` policy class node.
graph.assign(accountsID, OA), rbacID, PC));


// 8. Assign the object, `o1`, to the `Accounts` object attribute.
graph.assign(objectID, O), accountsID, OA));


// 9. Assign `u1` to the `Teller` user attribute and `u2` to the `Auditor` user attribute.
graph.assign(user1ID, U), tellerID, UA));
graph.assign(user2ID, U), auditorID, UA));


// 10. Create the associations for `Teller` and `Auditor` on `Account` in RBAC. `Teller` has read and write permissions, while `Auditor` just has read permissions.
graph.associate(tellerID, UA), accountsID, OA), new HashSet<>(Arrays.asList("r", "w")));
graph.associate(auditorID, UA), accountsID, OA), new HashSet<>(Arrays.asList("r")));


// 11. Create the `Branches` policy class.
long branchesID = graph.createNode(rand.nextLong(), "branches", PC, null));


// 12. Create an object attribute for `Branch 1`.
long branch1OAID = graph.createNode(rand.nextLong(), "branch 1", OA, null));

// 13. Assign the branch 1 OA to the branches PC
graph.assign(branch1OAID, OA), branchesID, PC));


// 14. Create the `Branch 1` user attribute
long branches1UAID = graph.createNode(rand.nextLong(), "branch 1", UA, null));


// 15. Assign the object, `o1`, to the `Branch 1` object attribute
graph.assign(objectID, O), branch1OAID, OA));


// 16. Assign the users, `u1` and `u2`, to the branch 1 user attribute
graph.assign(user1ID, U), branches1UAID, UA));
graph.assign(user2ID, U), branches1UAID, UA));


// 17. Create an association between the `branch 1` user attribute and the `branch 1` object attribute.
//This will give both users read and write on `o1` under the `branches` policy class.
graph.associate(branches1UAID, UA), branch1OAID, OA), new HashSet<>(Arrays.asList("r", "w")));


// 18. Test the configuration using the `PReviewDecider` implementation of the `Decider` interface.
//The constructor for a `PReviewDecider` receives the graph we created and a list of prohibitions.
//Since no prohibitions are used in this example, we'll pass null.
Decider decider = new PReviewDecider(graph);


// 19. Check that `u1` has read and write permissions on `o1`.
Set<String> permissions = decider.listPermissions(user1ID, 0, objectID);
assertTrue(permissions.contains("r"));
assertTrue(permissions.contains("w"));


// 20. Check that `u1` has read permissions on `o1`.
permissions = decider.listPermissions(user2ID, 0, objectID);
assertTrue(permissions.contains("r"));
```

#### Visualization
Below is a visual representation of the graph created in the bank teller example.
[![alt text](images/bankteller.png "bank teller example")](images/bankteller.png)


### Employee Record

#### Example configuration summary

- One policy class
- Users: bob, alice, charlie
- The objects are bob's and alice's name, salary, and ssn.
- All users are assigned to the Staff user attribute
- The Staff user attribute has read permissions on Public Info, which in this case is names.
- Charlie has the HR attribute
- HR has read and write permissions on Salaries and SSNs
- Bob and Alice have the Grp1Mgr and Grp2Mgr attributes, respectively
- Grp1Mgr and Grp2Mgr have read permissions on Grp1Salaries and Grp2Salaries, respectively
- Bob and Alice have read and write permissions on their name and ssn, and read permissions on their salaries. 

#### Access control state

- Alice can read and write her name and SSN, and read her salary, and the salaries of those in Group 2.
- Bob can read and write his name and SSN, and read his salary, and salaries of those in Group 1.
- Charlie can read and write all salaries and SSNs, and read all names.

```
Graph graph = new MemGraph();

// create nodes
// object attributes
long salariesID = graph.createNode(rand.nextLong(), "Salaries", OA, null));
long ssnsID = graph.createNode(rand.nextLong(), "SSNs", OA, null));
long grp1SalariesID = graph.createNode(rand.nextLong(), "Grp1 Salaries", OA, null));
long grp2SalariesID = graph.createNode(rand.nextLong(), "Grp2 Salaries", OA, null));
long publicID = graph.createNode(rand.nextLong(), "Public Info", OA, null));

long bobRecID = graph.createNode(rand.nextLong(), "Bob Record", OA, null));
long bobRID = graph.createNode(rand.nextLong(), "Bob r", OA, null));
long bobRWID = graph.createNode(rand.nextLong(), "Bob r/w", OA, null));

long aliceRecID = graph.createNode(rand.nextLong(), "Alice Record", OA, null));
long aliceRID = graph.createNode(rand.nextLong(), "Alice r", OA, null));
long aliceRWID = graph.createNode(rand.nextLong(), "Alice r/w", OA, null));

// objects for bob's name, salary, and ssn
long bobNameID = graph.createNode(rand.nextLong(), "bob name", O, null));
long bobSalaryID = graph.createNode(rand.nextLong(), "bob salary", O, null));
long bobSSNID = graph.createNode(rand.nextLong(), "bob ssn", O, null));

// objects for alice's name, salary, and ssn
long aliceNameID = graph.createNode(rand.nextLong(), "alice name", O, null));
long aliceSalaryID = graph.createNode(rand.nextLong(), "alice salary", O, null));
long aliceSSNID = graph.createNode(rand.nextLong(), "alice ssn", O, null));

// user attributes
long hrID = graph.createNode(rand.nextLong(), "HR", UA, null));
long grp1MgrID = graph.createNode(rand.nextLong(), "Grp1Mgr", UA, null));
long grp2MgrID = graph.createNode(rand.nextLong(), "Grp2Mgr", UA, null));
long staffID = graph.createNode(rand.nextLong(), "Staff", UA, null));
long bobUAID = graph.createNode(rand.nextLong(), "Bob", UA, null));
long aliceUAID = graph.createNode(rand.nextLong(), "Alice", UA, null));

// users
long bobID = graph.createNode(rand.nextLong(), "bob", U, null));
long aliceID = graph.createNode(rand.nextLong(), "alice", U, null));
long charlieID = graph.createNode(rand.nextLong(), "charlie", U, null));

// policy class
long pcID = graph.createNode(rand.nextLong(), "Employee Records", PC, null));


// assignments
// assign users to user attributes
graph.assign(charlieID, U), hrID, UA));
graph.assign(bobID, U), grp1MgrID, UA));
graph.assign(aliceID, U), grp2MgrID, UA));
graph.assign(charlieID, U), staffID, UA));
graph.assign(bobID, U), staffID, UA));
graph.assign(aliceID, U), staffID, UA));
graph.assign(bobID, U), bobUAID, UA));
graph.assign(aliceID, U), aliceUAID, UA));

// assign objects to object attributes
// salary objects
graph.assign(bobSalaryID, O), salariesID, OA));
graph.assign(bobSalaryID, O), grp1SalariesID, OA));
graph.assign(bobSalaryID, O), bobRID, OA));

graph.assign(aliceSalaryID, O), salariesID, OA));
graph.assign(aliceSalaryID, O), grp2SalariesID, OA));
graph.assign(aliceSalaryID, O), aliceRID, OA));

// ssn objects
graph.assign(bobSSNID, O), ssnsID, OA));
graph.assign(bobSSNID, O), bobRWID, OA));

graph.assign(aliceSSNID, O), aliceID, OA));
graph.assign(aliceSSNID, O), aliceRWID, OA));

// name objects
graph.assign(bobNameID, O), publicID, OA));
graph.assign(bobNameID, O), bobRWID, OA));

graph.assign(aliceNameID, O), publicID, OA));
graph.assign(aliceNameID, O), aliceRWID, OA));

// bob and alice r/w containers to their records
graph.assign(bobRID, OA), bobRecID, OA));
graph.assign(bobRWID, OA), bobRecID, OA));

graph.assign(aliceRID, OA), aliceRecID, OA));
graph.assign(aliceRWID, OA), aliceRecID, OA));


// assign object attributes to policy classes
graph.assign(salariesID, OA), pcID, PC));
graph.assign(ssnsID, OA), pcID, PC));
graph.assign(grp1SalariesID, OA), pcID, PC));
graph.assign(grp2SalariesID, OA), pcID, PC));
graph.assign(publicID, OA), pcID, PC));
graph.assign(bobRecID, OA), pcID, PC));
graph.assign(aliceRecID, OA), pcID, PC));

// associations
Set<String> rw = new HashSet<>(Arrays.asList("r", "w"));
Set<String> r = new HashSet<>(Arrays.asList("r"));

graph.associate(hrID, UA), salariesID, OA), rw);
graph.associate(hrID, UA), ssnsID, OA), rw);
graph.associate(grp1MgrID, UA), grp1SalariesID, OA), r);
graph.associate(grp2MgrID, UA), grp2SalariesID, OA), r);
graph.associate(staffID, UA), publicID, OA), r);
graph.associate(bobUAID, UA), bobRWID, OA), rw);
graph.associate(bobUAID, UA), bobRID, OA), r);
graph.associate(aliceUAID, UA), aliceRWID, OA), rw);
graph.associate(aliceUAID, UA), aliceRID, OA), r);

// test configuration
// create a decider
// not using prohibitions in this example, so null is passed
Decider decider = new PReviewDecider(graph, null);

// user: bob
// target: 'bob ssn'
// expected: [r, w]
// actual: [r, w]
Set<String> permissions = decider.listPermissions(bobID, 0, bobSSNID);
assertTrue(permissions.contains("r"));
assertTrue(permissions.contains("w"));

// user: bob
// target: 'bob ssn'
// expected: [r]
// actual: [r]
permissions = decider.listPermissions(bobID, 0, bobSalaryID);
assertTrue(permissions.contains("r"));

// user: bob
// target: 'alice ssn'
// expected: []
// actual: []
permissions = decider.listPermissions(bobID, 0, aliceSSNID);
assertTrue(permissions.isEmpty());

// user: bob
// target: 'alice salary'
// expected: []
// actual: []
permissions = decider.listPermissions(bobID, 0, aliceSalaryID);
assertTrue(permissions.isEmpty());

// user: bob
// target: 'bob ssn'
// expected: [r, w]
// actual: [r, w]
permissions = decider.listPermissions(aliceID, 0, aliceSSNID);
assertTrue(permissions.contains("r"));
assertTrue(permissions.contains("w"));

// user: charlie
// target: 'alice salary'
// expected: [r, w]
// actual: [r, w]
permissions = decider.listPermissions(charlieID, 0, aliceSalaryID);
assertTrue(permissions.contains("r"));
assertTrue(permissions.contains("w"));
```

#### Visualization
Below is a visual representation of the graph created in the employee record example.
[![alt text](images/emprec.png "employee record example")](images/emprec.png)

### Audit
#### Explain
Using the bank teller example described [above](#bank-teller), `auditor.explain(user1ID, objectID)` will result in:
```
RBAC
	u1-Teller-[r,w]-Accounts-o1
branches
	u1-branch 1-[r,w]-branch 1-o1
```
1. `u1` to `o1` via an association `Teller --[r, w]--> Accounts` under `RBAC`
2. `u1` to `o1` via an association `branch 1 --[r, w]--> branch 1` under `branches`

From the returned paths we can deduce that `u1` has `r, w` on `o1`.
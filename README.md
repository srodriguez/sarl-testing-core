# SARL Testing Framework

== Beta release ==

This project aims to provide a testing framework for agents developed with the [SARL Programming Language](http://www.sarl.io).

A simple example of use can be found in [this demo repository](https://github.com/srodriguez/sarl-testing-demo). TODO: Update with io.sarl.testing.basicexample

== How to ==

=== Setting up an agent ===
When testing the aspects of the system related to agent orientation, a mock agent is required. Estabishing this is simple; in the testing class, declare:

```
var agt : TestingAgent
```

=== Attaching skills and behaviors ===
To test any skills or behaviors they need to be attached to the agent. This is done by calling `withSkill` and `withBehavior`.

For attaching skills we first need to create a mock of them. This is done in the standard manner, by spying on the skill:

```
val RobotCtrlSkillMock = spy(RobotCtrlSkill)
```

With this done we can now attach the skill to the agent:
```
agt.withSkill(RobotCtrlSkillMock, RobotCtrl)
```

Note that only the Skill needs to be mocked; the capacity is used as-is.

Behaviors are simpler; like capacities they are attached without needing to be mocked:
```
agt.withBehavior(Robot)
```

=== Testing aspects ===
With the agt initialized and relevant skills and behaviors attached we are now able to create the actual tests.

For example, to test what a skill's function returns we can call the function through the mock so long as it is attached to the agent:
```	
@Test
def battery_level_initializes_as_medium {
	assertEquals(RobotCtrlSkillMock.battery_level, BatteryLevel.MED)
}
```

Alternatively, when using the Goal engine if we want to test whether a plan has been correctly rated under a specific goal then we can attach the mocked components of the goal engine and create that specific goal, allowing us to test how the system will react:

```
@Test
def rates_plans_appropriately {

	agt.withSkill(PlanSelectionConstraintsMock, PlanSelectionConstraints)
	agt.withBehavior(Robot)

	// When we are trying to go to location 3, 4
	val g34 = new Goto(new Location(3, 4))

	doReturn(true).when(this.RobotCtrlSkillMock).obstacles(any(Location))

	val list = PlanSelectionConstraintsMock.applicablePlans(g34).sortBy[ap|1 - ap.rating]

	// Then Plan_Around should be our first plan, and Plan_Strait our second plan
	assertNotNull(list)
	assertEquals(2, list.size)

	assertEquals(Plan_Around, list.get(0).plan)
	assertEquals(Plan_Strait, list.get(1).plan)
	
}
```

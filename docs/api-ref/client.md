# The Client API

<!--- WARNING: THIS FILE WAS AUTOGENERATED! DO NOT EDIT! Instead, edit the notebook w/the location & name as this file.-->

The client API allows you to inspect data and artifacts from your flows.


<DocSection type="class" name="Flow" module="metaflow" heading_level="3" link="https://github.com/Netflix/metaflow/tree/master/metaflow/client/core.py#L1671">
<SigArgSection>
<SigArg name="*args" /><SigArg name="**kwargs" />
</SigArgSection>
<Description summary="A Flow represents all existing flows with a certain name, in other words,\nclasses derived from 'FlowSpec'" extended_summary="As such, it contains all Runs (executions of a flow) related to this flow." />
<ParamSection name="Attributes">
	<Parameter name="latest_run" type="Run" desc="Latest Run (in progress or completed, successfully or not) of this Flow" />
	<Parameter name="latest_successful_run" type="Run" desc="Latest successfully completed Run of this Flow" />
</ParamSection>
</DocSection>



<DocSection type="class" name="Run" module="metaflow" heading_level="3" link="https://github.com/Netflix/metaflow/tree/master/metaflow/client/core.py#L1512">
<SigArgSection>
<SigArg name="pathspec" default="None" /><SigArg name="attempt" default="None" /><SigArg name="_object" default="None" /><SigArg name="_parent" default="None" /><SigArg name="_namespace_check" default="True" />
</SigArgSection>
<Description summary="A Run represents an execution of a Flow" extended_summary="As such, it contains all Steps associated with the flow." />
<ParamSection name="Attributes">
	<Parameter name="data" type="MetaflowData" desc="Container of all data artifacts produced by this run" />
	<Parameter name="successful" type="boolean" desc="True if the run successfully completed" />
	<Parameter name="finished" type="boolean" desc="True if the run completed" />
	<Parameter name="finished_at" type="datetime" desc="Time this run finished" />
	<Parameter name="code" type="MetaflowCode" desc="Code package for this run (if present)" />
	<Parameter name="end_task" type="Task" desc="Task for the end step (if it is present already)" />
</ParamSection>
</DocSection>



<DocSection type="class" name="Step" module="metaflow" heading_level="3" link="https://github.com/Netflix/metaflow/tree/master/metaflow/client/core.py#L1374">
<SigArgSection>
<SigArg name="pathspec" default="None" /><SigArg name="attempt" default="None" /><SigArg name="_object" default="None" /><SigArg name="_parent" default="None" /><SigArg name="_namespace_check" default="True" />
</SigArgSection>
<Description summary="A Step represents a user-defined Step (a method annotated with the @step decorator)." extended_summary="As such, it contains all Tasks associated with the step (ie: all executions of the\nStep). A linear Step will have only one associated task whereas a foreach Step will have\nmultiple Tasks." />
<ParamSection name="Attributes">
	<Parameter name="task" type="Task" desc="Returns a Task object from the step" />
	<Parameter name="finished_at" type="datetime" desc="Time this step finished (time of completion of the last task)" />
	<Parameter name="environment_info" type="Dict" desc="Information about the execution environment (for example Conda)" />
</ParamSection>
</DocSection>



<DocSection type="class" name="Task" module="metaflow" heading_level="3" link="https://github.com/Netflix/metaflow/tree/master/metaflow/client/core.py#L896">
<SigArgSection>
<SigArg name="*args" /><SigArg name="**kwargs" />
</SigArgSection>
<Description summary="A Task represents an execution of a step." extended_summary="As such, it contains all data artifacts associated with that execution as\nwell as all metadata associated with the execution.\n\nNote that you can also get information about a specific *attempt* of a\ntask. By default, the latest finished attempt is returned but you can\nexplicitly get information about a specific attempt by using the\nfollowing syntax when creating a task:\n`Task('flow/run/step/task', attempt=<attempt>)`. Note that you will not be able to\naccess a specific attempt of a task through the `.tasks` method of a step\nfor example (that will always return the latest attempt)." />
<ParamSection name="Attributes">
	<Parameter name="metadata" type="List[Metadata]" desc="List of all metadata associated with the task" />
	<Parameter name="metadata_dict" type="Dict" desc="Dictionary where the keys are the names of the metadata and the value are the values\nassociated with those names" />
	<Parameter name="data" type="MetaflowData" desc="Container of all data artifacts produced by this task" />
	<Parameter name="artifacts" type="MetaflowArtifacts" desc="Container of DataArtifact objects produced by this task" />
	<Parameter name="successful" type="boolean" desc="True if the task successfully completed" />
	<Parameter name="finished" type="boolean" desc="True if the task completed" />
	<Parameter name="exception" type="object" desc="Exception raised by this task if there was one" />
	<Parameter name="finished_at" type="datetime" desc="Time this task finished" />
	<Parameter name="runtime_name" type="string" desc="Runtime this task was executed on" />
	<Parameter name="stdout" type="string" desc="Standard output for the task execution" />
	<Parameter name="stderr" type="string" desc="Standard error output for the task execution" />
	<Parameter name="code" type="MetaflowCode" desc="Code package for this task (if present)" />
	<Parameter name="environment_info" type="Dict" desc="Information about the execution environment (for example Conda)" />
</ParamSection>
</DocSection>

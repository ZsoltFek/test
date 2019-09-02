Documentation for GitLabSkill

Interaction is possible after calling the invocation name, which is itware gitlab, this sends the LaunchRequest the skill requires

Between the LaunchRequest and SessionEndedRequest (which marks the end of the user interaction session), session attributes are kept (later: attributes), which are the following:
    -projectID: allows seamless interaction with skill as long as user intends to give requests related to the same project (switching to a different project is possible via ProjectSwitchIntentHandler)
    -previousIntentType: every time an intent handler is called, the intent's name is kept as this attribute, this helps HelpIntentHandler to give the user syntax support based on the latest attempted request

Each user request is evaluated by the Alexa skill and assigned to an intent based on the set sample utterances. 
If the user's input matches any sample utterance set in the skill, the intent connected to that utterance will be called (this is done via the canHandle() function in each handler).
Intents may have words and slots. Slots allow user input to be parsed and used later on in the code. If the values expected in a slot cannot be matched to one of Alexa's built-in slots, its possible values must be given.

Each IntentHandler checks the called intent's type, and if it matches the handler's type, its handle() function is called with the input generated from the user's request (handlerInput) as its argument.

At the end of the handle() function, an Alexa output is generated via the built-in responseBuilder (handlerInput.responseBuilder).
    - .speak(argument) prompts alexa to repond to the request with the argument
    - .reprompt(argument) prompts alexa to set the reprompt text to argument, which is sent as a reminder if the user doesn't continue the interaction session
    - .getResponse() waits for the user's next request

handlerInput is destructured for easier access to its most used values
    -handlerInput.requestEnvelope.request.intent.slots provides access to the slot values the user has given in the request
        -values can be accessed via slots.SlotName.value afterwards
    -handlerInput.attributesManager provides access to the interaction session's stored session attributes
        -attributes can be accessed via attributesManager.getSessionAttributes() and set via attributesManager.setSessionAttributes(argument)
    -handlerInput.responseBuilder provides access to the skill's reponse builder through which Alexa responses may be generated

The following intents have been implemented as of 2019.08.14. (for sample utterances see skill_build.json "samples" array in a given intent's object):
    -ProjectQueryIntent
        when called, reponds with the current project's name
    -ProjectSwitchIntent
        when called, switches to the project indicated by the user in the {ProjectNameSlot} slot
    -JobCheckIntent
        when called, lists information about the current project's jobs
        {JobType} slot is optional, allows user to check only jobs with a given status
    -CoverageCheckIntent
        when called, provides coverage information on the current project's pipelines
    -PipelineCheckIntent
        when called, lists information about the current project's pipelines
        {PipelineSlotType} slot is optional, allows user to check only pipelines with a given status
    -ScheduleCheckIntent
        when called, lists information about the current project's pipeline schedules
    -ScheduleChangeIntent
        when called, applies the change requested by the user to the indicated pipeline schedules
        possible changes:
            -{BranchSlot} allows user to edit which branch the schedule should be run
            -{TimeSlot} allows user to edit at what time of day the schedule should be run
            -{DaySlot} allows user to edit which specific day the schedule should be run (built-in amazon slot type is used, valus may only be used for one-time schedules)
            -{SpecialDaySlot} allows user to create regular pipeline schedule
                notes:
                {DaySlot} and {SpecialDaySlot} are interchangeable in user requests, 
                {SpecialDaySlot} is a custom slot type with values such as "every monday" and "wednesdays"
                {DaySlot} is a built-in slot type, translates user input such as "next tuesday" or "first of november" to specific date
                the values of {TimeSlot} and {DaySlot} or {SpecialDaySlot} create the cron format string passed to the API
                all slots are optional, only the filled slots will apply changes to the schedule, rest remain unchanged
    -ScheduleCreateIntent
        when called, creates a pipeline schedule based on user's input
             slots are {BranchSlot}, {TimeSlot}, {DaySlot} and {SpecialDaySlot}, same logic as in ScheduleChangeIntent
             in a schedule creation request all slots are mandatory (either {DaySlot} or {SpecialDaySlot} must be filled)
    -ScheduleDeleteIntent
        when called, deletes indicated pipeline schedule
        {ScheduleSlot} slot is mandatory, contains the to-be-deleted pipeline schedule's ID
    -MRCheckIntent
        when called, lists information about the current project's merge requests
    -MRChangeIntent
        when called, applies the change request by the user to the indicated merge request
        possible changes:
            -{TargetBranchSlot} the branch's name onto which the merge should happen
            -{AssigneeSlot} the user's name who is to be assigned to the merge request
                notes:
                both are custom slot types, all possible values must be uploaded into them
                all slots are optional, only the filled slots will apply changes to the merge request, rest remain unchanged
    -MRCreateIntent
        when called, creates merge request based on user's input
        slots are {SourceBranchSlot}, the source branch's name and {TargetBranchSlot}, the target branch's name, both are mandatory
    -MRDeleteIntent
        when called, deletes indicated merge request
        {MRSlot} slot is mandatory, contains the to-be-deleted merge request's ID
    -FeatureListIntent
        when called, provides a list of all features with a description and sample utterance how to call them



Adding new intents:
    -new intent must be created in on the skill's developer.amazon.com page with slots and sample utterances
    -new handler must be created in the index.js file, the mandatory build of it is as follows:
    const IntentNameHandler = {
        canHandle(handlerInput) {
            return handlerInput.requestEnvelope.request.type === "IntentRequest" && (handlerInput.requestEnvelope.request.intent.name === "IntentName");
        },
        handle(handlerInput) {
            return handlerInput.responseBuilder.speak(speakOutput).reprompt(repromptOutput).getResponse();
        },
    };
    -new intent's handler must be added to exports.handler list via skillBuilder.addRequestHandler()
        multiple can be added via the skillBuilder.addRequestHandlers() function, see the end of index.js for current list

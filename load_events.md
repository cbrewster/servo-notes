Servo `load` events for iframes
---

Starting point:
```
[no src load event]:
--------------------
htmliframeelement: bind_to_tree
htmliframeelement: process_the_iframe_attributes (FirstTime, no src)
htmliframeelement: IframeLoadEventSteps (runnable) -> event_loop
htmliframeelement: dispatch load event

[initial about:blank]:
----------------------
htmliframeelement: bind_to_tree
htmliframeelement: create_nested_browsing_context
htmliframeelement: navigate_or_reload_child_browsing_context (InitialAboutBlank)
script_thread: process_attach_layout
script_thread: handle_new_layout
script_thread: start_page_load_about_blank
parser_context: process_response_eof
document: finish_load
document: DocumentLoadsComplete -> event_loop
script_thread: handle_loads_complete
script_thread: DocumentProgressHandler (runnable) -> event_loop
document: dispatch load event

[normal load]:
--------------
htmliframeelement: bind_to_tree or src mutate
htmliframeelement: navigate_or_reload_child_browsing_context (Normal)
htmliframeelement: ScriptLoadedURLInIFrame -> constellation
constellation: handle_script_loaded_url_in_iframe_msg
constellation: push pending frame change
constellation: new_pipeline
pipeline: spawn
pipeline: AttachLayout -> event_loop
script_thread: handle_new_layout
script_thread: start_page_load or start_page_load_about_blank
script_thread: Fetch
resource_threads: fetch
fetch/methods: fetch
fetch/methods: fetch_with_cors_cache
fetch/methods: main_fetch
net_traits: process_response_eof
servoparser: process_response_eof
document: finish_load
document: DocumentLoadsComplete -> event_loop
script_thread: handle_loads_complete
script_thread: DocumentProgressHandler (runnable) -> event_loop
document: dispatch load event
```
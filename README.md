flowchart TD
    subgraph S1 ["Stage 1: Ingestion & Setup"]
        A["Staging Table<br/>(action_config_staging)"] -->|sp_load_action_config| B["Core Tables<br/>(actions, actions_details, actions_links)"]
        B --> C["Assignments & Subscriptions<br/>(action_assignments, actions_subscription_config)"]
    end

    subgraph S2 ["Stage 2: Execution & Assignment"]
        C -->|sp_process_ui_action /<br/>sp_process_data_action| D["Insert actions_execution<br/>(status = 'Active')"]
        D --> E1["Insert actions_bell<br/>(bell_type = 'Action Assigned')"]
        D --> E2["Insert actions_email<br/>(email_sent_at = NULL)"]
    end

    subgraph S3 ["Stage 3: Refresh & Escalation"]
        D -->|sp_refresh_actions_detail| F{"Due date /<br/>Escalation passed?"}
        F -- "Yes" --> G["Update actions_execution<br/>(is_alert = TRUE)"]
        G --> H1["actions_bell ('Alert Escalation')"]
        G --> H2["actions_email ('Alert Escalation')"]
    end

    subgraph S4 ["Stage 4: Resolution"]
        D -->|sp_resolve_ui_action| I["Update actions_execution<br/>(status = 'Resolved')"]
        I --> J1["actions_bell ('Action Resolved')"]
        I --> J2["actions_email ('Action Resolved')"]
    end

    subgraph S5 ["Stage 5: User Interaction & Consumption"]
        E1 & H1 & J1 --> K["fn_get_bell_icon_notification_in_app<br/>(UI Bell Icon Dropdown)"]
        K --> L["User Reads/Clears<br/>(sp_ui_bell_notification_override)"]
        E2 & H2 & J2 --> M["Backend Email Worker<br/>(Sends Email & Updates email_sent_at)"]
    end

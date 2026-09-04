{
    "title": "SSH Brute Force Detection Dashboard",
    "description": "",
    "inputs": {
        "input_global_trp": {
            "options": {
                "defaultValue": "0,",
                "token": "global_time"
            },
            "title": "Global Time Range",
            "type": "input.timerange"
        }
    },
    "defaults": {
        "dataSources": {
            "ds.search": {
                "options": {
                    "queryParameters": {
                        "earliest": "$global_time.earliest$",
                        "latest": "$global_time.latest$"
                    }
                }
            },
            "ds.spl2": {
                "options": {
                    "queryParameters": {
                        "earliest": "$global_time.earliest$",
                        "latest": "$global_time.latest$"
                    }
                }
            }
        },
        "visualizations": {
            "global": {
                "showProgressBar": true
            }
        }
    },
    "visualizations": {
        "viz_ATfhhLjz": {
            "containerOptions": {},
            "context": {},
            "dataSources": {
                "primary": "ds_f3t0pSfB"
            },
            "options": {},
            "showLastUpdated": false,
            "showProgressBar": true,
            "title": "Failed logins over time",
            "type": "splunk.line"
        },
        "viz_GXPfQDTe": {
            "containerOptions": {},
            "context": {},
            "dataSources": {
                "primary": "ds_d2O2Rmc6"
            },
            "options": {},
            "showLastUpdated": false,
            "showProgressBar": true,
            "title": "Event type breakdown",
            "type": "splunk.pie"
        },
        "viz_dFDaft0f": {
            "containerOptions": {},
            "context": {},
            "dataSources": {
                "primary": "ds_5UOAh9ic"
            },
            "options": {},
            "showLastUpdated": false,
            "showProgressBar": true,
            "title": "Failed logins by IP",
            "type": "splunk.bar"
        },
        "viz_nIP7HaQV": {
            "containerOptions": {},
            "context": {},
            "dataSources": {
                "primary": "ds_DCS3qHfV"
            },
            "options": {},
            "showLastUpdated": false,
            "showProgressBar": true,
            "title": "Top offending IP (single value)",
            "type": "splunk.area"
        },
        "viz_vFkLMgqW": {
            "dataSources": {
                "primary": "ds_eIAUQcm4"
            },
            "options": {
                "count": 20,
                "dataOverlayMode": "none",
                "drilldown": "none",
                "showInternalFields": false,
                "showRowNumbers": false
            },
            "title": "Top targeted usernames",
            "type": "splunk.table"
        }
    },
    "dataSources": {
        "ds_3qiy0XmV": {
            "name": "Failed logins over time search",
            "options": {
                "query": "source=\"ssh_logs_new.json\" host=\"Mahhyya\" sourcetype=\"_json\" event_type=\"Failed SSH Login\" | timechart span=1h count",
                "queryParameters": {
                    "earliest": "0",
                    "sampleRatio": 1
                }
            },
            "type": "ds.search"
        },
        "ds_5UOAh9ic": {
            "name": "Failed logins by IP search",
            "options": {
                "query": "source=\"ssh_logs_new.json\" host=\"Mahhyya\" sourcetype=\"_json\" event_type=\"Failed SSH Login\" | stats count as total_attempts by id.orig_h | where total_attempts >= 5 | sort -total_attempts",
                "queryParameters": {
                    "earliest": "0",
                    "sampleRatio": 1
                }
            },
            "type": "ds.search"
        },
        "ds_DCS3qHfV": {
            "name": "Top offending IP _single value_ search",
            "options": {
                "query": "source=\"ssh_logs_new.json\" host=\"Mahhyya\" sourcetype=\"_json\" event_type=\"Failed SSH Login\" | stats count as total_attempts by id.orig_h | sort -total_attempts | head 10",
                "queryParameters": {
                    "earliest": "0",
                    "sampleRatio": 1
                }
            },
            "type": "ds.search"
        },
        "ds_d2O2Rmc6": {
            "name": "Event type breakdown search",
            "options": {
                "query": "source=\"ssh_logs_new.json\" host=\"Mahhyya\" sourcetype=\"_json\" | stats count by event_type",
                "queryParameters": {
                    "earliest": "0",
                    "sampleRatio": 1
                }
            },
            "type": "ds.search"
        },
        "ds_eIAUQcm4": {
            "name": "Top targeted usernames search",
            "options": {
                "query": "source=\"ssh_logs_new.json\" host=\"Mahhyya\" sourcetype=\"_json\" event_type=\"Failed SSH Login\" | top limit=10 username",
                "queryParameters": {
                    "earliest": "0",
                    "sampleRatio": 1
                }
            },
            "type": "ds.search"
        },
        "ds_f3t0pSfB": {
            "name": "Failed logins over time search 2",
            "options": {
                "query": "source=\"ssh_logs_new.json\" host=\"Mahhyya\" sourcetype=\"_json\" event_type=\"Failed SSH Login\" | timechart span=1h count",
                "queryParameters": {
                    "earliest": "0",
                    "sampleRatio": 1
                }
            },
            "type": "ds.search"
        }
    },
    "layout": {
        "globalInputs": [
            "input_global_trp"
        ],
        "layoutDefinitions": {
            "layout_1": {
                "options": {
                    "height": 960,
                    "width": 1440
                },
                "structure": [
                    {
                        "item": "viz_vFkLMgqW",
                        "position": {
                            "h": 450,
                            "w": 720,
                            "x": 0,
                            "y": 0
                        },
                        "type": "block"
                    },
                    {
                        "item": "viz_dFDaft0f",
                        "position": {
                            "h": 450,
                            "w": 720,
                            "x": 720,
                            "y": 0
                        },
                        "type": "block"
                    },
                    {
                        "item": "viz_nIP7HaQV",
                        "position": {
                            "h": 600,
                            "w": 720,
                            "x": 720,
                            "y": 450
                        },
                        "type": "block"
                    },
                    {
                        "item": "viz_GXPfQDTe",
                        "position": {
                            "h": 600,
                            "w": 720,
                            "x": 0,
                            "y": 450
                        },
                        "type": "block"
                    },
                    {
                        "item": "viz_ATfhhLjz",
                        "position": {
                            "h": 450,
                            "w": 1440,
                            "x": 0,
                            "y": 1050
                        },
                        "type": "block"
                    }
                ],
                "type": "grid"
            }
        },
        "options": {},
        "tabs": {
            "items": [
                {
                    "label": "Overview",
                    "layoutId": "layout_1"
                }
            ]
        }
    },
    "applicationProperties": {}
}

---
title: Action
---
import UniversalMap from '/docs/images/universal_map/_universal_map.mdx';
import TechnicalTag from '../term_tags/_tag.mdx';
import SetupHeader from '../images/universal_map/_um_setup_header.mdx'
import ConnectHeader from '../images/universal_map/_um_connect_header.mdx';
import CreateHeader from '../images/universal_map/_um_create_header.mdx';
import ValidateHeader from '../images/universal_map/_um_validate_header.mdx';

<UniversalMap setup='inactive' connect='inactive' create='inactive' validate='active'/> 

## Overview

### Definition

An Action is a Python class with a `run()` method that takes a <TechnicalTag relative="../" tag="validation_result" text="Validation Result" /> and does something with it.

### Features and promises

Actions are highly customizable.  Great Expectations comes with common Actions for such things as sending email or Slack notifications, updating <TechnicalTag relative="../" tag="data_docs" text="Data Docs" />, and storing <TechnicalTag relative="../" tag="validation_result" text="Validation Results" /> out of the box.  However, it is easy to create custom Actions by creating a subclass of Great Expectations' `ValidationAction` class and overwriting it's `_run()` method. This means that you can configure an Action to do literally anything you are capable of programming in Python in response to a <TechnicalTag relative="../" tag="checkpoint" text="Checkpoint" /> <TechnicalTag relative="../" tag="validation" text="Validation" /> completing.

### Relationship to other objects

Actions are executed by <TechnicalTag relative="../" tag="validation_operator" text="Validation Operators" />, which are a component of Checkpoints.  In general, Validation Operators function behind the scenes, so it is enough to know that Actions are configured inside the `action_list` parameter of a Checkpoint's configuration, and execute every time the Checkpoint finishes running a Validation.

## Use cases

Configuring, implementing, and executing an Action (custom or otherwise) takes place in the Validate Data step. Creating custom Actions is a process that falls outside the workflow of using Great Expectations.

<ValidateHeader/>

Actions are configured when Checkpoints are created in the Validate Data step of working with Great Expectations.  After that, Checkpoints that have a populated `action_list` in their configuration will execute the indicated Actions every time they finish running a Validation.

## Features

### Versatility

The features of a specific Action depend on what that Action is designed to do.  An Action can perform anything that can be done with Python code, making them phenomenally versatile.

### Convenience

Since some Actions are common, Great Expectations implements them out of the box: You don't have to write every Action as a custom one.  These Actions are subclasses of the `ValidationAction` class, and you can view them in the [great_expectations.checkpoint.actions](https://github.com/great-expectations/great_expectations/blob/develop/great_expectations/checkpoint/actions.py) module.

For a quick overview, however, some of the available pre-built Actions are:

- `EmailAction`: sends an email to a given list of email addresses.
- `MicrosoftTeamsNotificationAction`: sends a Microsoft Teams notification to a given webhook.
- `SlackNotificationAction`: sends a Slack notification to a given webhook.
- `StoreEvaluationParametersAction`: extracts Evaluation Parameters from a Validation Result and stores them in the Store configured for this Action.
- `StoreMetricsAction`: extracts Metrics from a Validation Result and stores them in a Metrics Store.
- `StoreValidationResultAction`: stores a Validation Result in the `ValidationsStore`.
- `UpdateDataDocsAction`: notifies the site builders of all the data docs sites of the Data Context that a validation result should be added to the data docs.


## API basics

### How to access

Actions are not intended to be manually instantiated or accessed.  Instead, they are included in the `action_list` parameter of a Checkpoint's configuration, and when the Checkpoint finishes running a Validation it will then run the Actions in its `action_list` in order of appearance.

Classes that implement Actions can be found in the `great_expectations.checkpoint.actions` module, which you can view on GitHub:
- [great_expectations.checkpoint.actions](https://github.com/great-expectations/great_expectations/blob/develop/great_expectations/checkpoint/actions.py)

### How to create

Custom actions need to be created in Python code, and can be implemented as Plugins.  In order for a Python class to be a valid Action, it must conform to the Action API.  Specifically, it must implement a `run()` method which accepts three required parameters, two named optional parameters, and any number of `**kwargs.`

**Required parameters:**
- `validation_result_suite`: an instance of the `ExpectationSuiteValidationResult` class.
- `validation_result_suite_identifier`: an instance of either the `ValidationResultIdentifier` class (for open source Great Expectations) or the `GeCloudIdentifier` (from Great Expectations Cloud).
- `data_asset`: an instance of the `Validator` class.

**Optional parameters:**
- `expectation_suite_identifier`
- `checkpoint_identifier`

**Additional parameters:**
- **kwargs: named parameters that are specific to a given Action, and need to be assigned a value in the Action's configuration in a Checkpoint's `action_list`.

The required and optional named parameters will be automatically passed to the Action from the Checkpoint that the Action is included with, after the Checkpoint completes Validation.  This means you can configure your custom Actions to behave conditionally based on the Validation Results your Checkpoint generates, or the values passed along with any of those named parameters.  Additional `**kwargs` parameters can be included, but they cannot be passed automatically by the Checkpoint.  Therefore, you will have to specify the name and value for these parameters in the configuration for their Action, in the Checkpoint's `action_list`.

The best practice when creating a custom Action is to create a subclass of the `ValidationAction` class found in the `great_expectations.checkpoint.actions` module.  Leave the inherited `run()` method as its default, and overwrite the `_run()` method to contain your functionality.  There are numerous examples of this practice in the subclasses of `ValidationAction` located in the `great_expectations.checkpoint.actions` module, which you can view on GitHub:
- [great_expectations.checkpoint.actions](https://github.com/great-expectations/great_expectations/blob/develop/great_expectations/checkpoint/actions.py)

If you develop a custom Action, consider making it a contribution in the [Great Expectations open source GitHub project](https://github.com/great-expectations/great_expectations).  You can also [reach out to us on Slack](https://greatexpectations.io/slack) if you need additional guidance in your efforts.

### Configuration

Actions are configured inside of the `action_list` parameter for Checkpoints.  In general, you will need to provide at least a `name` (which is user defined and does not need to correspond to anything specific) and, in the in the parameters under `action` a `class_name` (which should correspond to the name of the Action's Python class).  If you are implementing a custom Action in a Plugin, you will also need to include a `module_name` in the parameters under `action` which references where your Plugin is located.  Any other keys placed under `action` will be passed to the Action's class as additional key word arguments.

Below is an example of an `action_list` configuration that performs some common Actions that are built in to the Great Expectations code base:

```yaml
action_list:
- name: store_validation_result
  action:
    class_name: StoreValidationResultAction
- name: store_evaluation_params
  action:
    class_name: StoreEvaluationParametersAction
- name: update_data_docs
  action:
    class_name: UpdateDataDocsAction
```

For more examples of how to configure Actions in a Checkpoint, please see our [how-to guides on Actions](../guides/validation/index.md#validation-actions).
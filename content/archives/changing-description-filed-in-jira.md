+++
title = "A little about Atlassian Jira and changing Description Field"
date = "2013-07-18"
tags = ["jira", "tips"]
type = "post"
+++

I finally decided to write a little about changing default description field in Jira.

A short task description.
I needed to add some entries in Description field in the opening new bug. This field is empty by default. Many people have this problem and there is [feature request](http://jira.atlassian.com/browse/JRA-4812) in Atlassian bug-tracker.


In principle, there is a particular article about [changing Description field](https://confluence.atlassian.com/display/JIRA/Setting+a+Default+Value+in+the+Description+Field) in the Atlassian wiki, but I decided to describe the process again, as soon as I read and could not understand how to properly fit the options. 

So, according the article, to achieve the desired results you need to edit the file `WEB-INF/classes/templates/jira/issue/field/description-edit.vm`. The location of it depends on how and where Jira to install. In my case, this location was `/opt/atlassian/jira/atlassian-jira/`.

Needs to make a backup of this file just in case.

Open file. Below shows the content: 

		#disable_html_escaping()
		#customControlHeader ($action $field.id $i18n.getText($field.nameKey) $fieldLayoutItem.required $displayParameters $auiparams)
		## setup some additional parameters
		$!rendererParams.put("class", "long-field")
		$!rendererParams.put("rows", "12")
		$!rendererParams.put("wrap", "virtual")
		#if ($mentionable)
		    $!rendererParams.put("mentionable", true)
		    #if ($issue.project.key && $issue.project.key != "")
		        $!rendererParams.put("data-projectkey", "$!issue.project.key")
		    #end
		    #if ($issue.key && $issue.key != "")
		        $!rendererParams.put("data-issuekey", "$!issue.key")
		    #end
		#end
		## let the renderer display the edit component
		$rendererDescriptor.getEditVM($!description, $!issue.key, $!fieldLayoutItem.rendererType, $!field.id, $field.name, $rendererParams, false)
		#customControlFooter ($action $field.id $fieldLayoutItem.getFieldDescription() $displayParameters $auiparams)

I did not immediately realize what a place to insert the desired value, and it is important to me whether it is a place. The result I was the following:


		#disable_html_escaping()
		#customControlHeader ($action $field.id $i18n.getText($field.nameKey) $fieldLayoutItem.required $displayParameters $auiparams)
		## setup some additional parameters
		$!rendererParams.put("class", "long-field")
		$!rendererParams.put("rows", "12")
		$!rendererParams.put("wrap", "virtual")
		#if ($mentionable)
		    $!rendererParams.put("mentionable", true)
		    #if ($issue.project.key && $issue.project.key != "")
		        $!rendererParams.put("data-projectkey", "$!issue.project.key")
		    #end
		    #if ($issue.key && $issue.key != "")
		        $!rendererParams.put("data-issuekey", "$!issue.key")
		    #end
		#end

		##### description
		#if ($issue.getIssueTypeObject().getName() == 'Bug')
		    #if($description == '')
		        #set ($description = '*Шаги:*\
		     -   ...\
		\
		*Результат:*\
		*Ожидаемый результат:*\')
		    #set ($description = $description.replace('\',' '))
		#end
		#end
		#####

		## let the renderer display the edit component
		$rendererDescriptor.getEditVM($!description, $!issue.key, $!fieldLayoutItem.rendererType, $!field.id, $field.name, $rendererParams, false)
		#customControlFooter ($action $field.id $fieldLayoutItem.getFieldDescription() $displayParameters $auiparams)

Entries is inserted between `#####`. 

In addition it should be said that the line `# if ($ issue.getIssueTypeObject (). GetName () == 'Bug')` lets you specify the desired type of problem for which the ticket is opened. In my case, change the description required only for "Bug", that indicated in the row.
After saving, you must restart the Jira instance for changes to take effect
Really, I had to do it a few times as I should have figured out the right amount of white space to achieve the desired indentation for the entries in Description field.

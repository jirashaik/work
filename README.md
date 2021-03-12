 //Read CSV from issue attachment
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.comments.CommentManager
import com.atlassian.jira.issue.AttachmentManager
import com.atlassian.jira.issue.attachment.Attachment
import com.atlassian.jira.issue.attachment.FileAttachments
import org.apache.commons.io.FilenameUtils
import com.atlassian.jira.util.io.InputStreamConsumer
import org.springframework.util.StreamUtils
import java.io.FileOutputStream
import java.io.InputStream
import java.io.IOException
import groovy.util.XmlParser
import groovy.util.XmlSlurper
import org.apache.log4j.Logger;
import org.apache.log4j.Level;
Logger logger = Logger.getLogger("test")
logger.setLevel(Level.DEBUG)
    
def issueManager = ComponentAccessor.getIssueManager()
def aM= ComponentAccessor.getAttachmentManager()
def pM = ComponentAccessor.getAttachmentPathManager()
def issue = issueManager.getIssueObject("CBT-61632")
def attachments = aM.getAttachments(issue)
 
String attachmentFileText=""
 
if (!attachments.isEmpty()) {
for (Attachment a in attachments) {
def fileName=FilenameUtils.getBaseName(a.getFilename())
//log.warn (fileName)
def fileExtension=FilenameUtils.getExtension(a.getFilename())
//log.warn (fileExtension)
    File attachmentFile = aM.streamAttachmentContent(a, new FileInputStreamConsumer(a.getFilename()))
logger.debug("file : " + "/n"+attachmentFile.text)
 
}
}
 
 
class FileInputStreamConsumer implements InputStreamConsumer{
 
private final String filename;
public FileInputStreamConsumer(String filename) {
this.filename = filename;
}
@Override
public File withInputStream(InputStream is) throws IOException {
final File f = File.createTempFile(FilenameUtils.getBaseName(filename), FilenameUtils.getExtension(filename));
StreamUtils.copy(is, new FileOutputStream(f));
return f;
}
}
////////////////////////////////////

script to remove link types which are not used on a project


<script type="text/javascript">
 
jQuery(document).ready(function($) {
 
//key is issue key
var key = document.querySelector('meta[name="ajs-issue-key"]').content
 
//if we get the key
if(key){
//split using - and get first element that is pkey
var pKey = key.split('-')[0]
 
//config is a map and it has key value pair, key has the project key and value is the array object which holds links that needs to be removed for //that project key
 
var config = new Map()
 
//setting the key value
config.set('CBT',['Enables','Enabled by'])
config.set('CT',['Enables','Enabled by'])
 
//checking if our config the current project key has any config or not
if(config.has(pKey)){
 
    var removeCTE = setInterval(function(){
 
    //we are getting elements to be removed and looping and removing
    var linksToBeRemoved = config.get(pKey)
 
 for(var i=0;i<linksToBeRemoved.length-1;i++){
    AJS.$("#link-type.select option[value ='"+linksToBeRemoved[i]+"']").remove();
    AJS.$("#issuelinks-linktype.select option[value ='"+linksToBeRemoved[i]+"']").remove();
     }
   }, 1000);
  }
 }
});
 
</script>

////////////////////////////////
 Script for pulling Filter details to check use of specific Workflow statuses:

//===================================================================================================
//Script for Getting filters list with owner information who are using these two workflow statuses.
//====================================================================================================
 
import com.atlassian.jira.bc.filter.SearchRequestService
import com.atlassian.jira.issue.search.SearchRequest
import com.atlassian.jira.bc.user.search.UserSearchParams
import com.atlassian.jira.bc.user.search.UserSearchService
import java.lang.StringBuilder
import com.atlassian.jira.user.ApplicationUser
import com.atlassian.jira.component.ComponentAccessor
 
import org.apache.log4j.Logger
import org.apache.log4j.Level
 
  
 
def logger = Logger.getLogger("testscript")
logger.setLevel(Level.DEBUG)
 
// statuses = the values I want to search for
String STATUS_ONE = 'As Is'
String STATUS_TWO = 'To Be'
 
//maxUsers must be set as per number of users in JIRA Instance setting it default to 10k
int maxUsers=10000
 
//initializing instance of SearchRequestService
SearchRequestService searchRequestService = ComponentAccessor.getComponent(SearchRequestService.class)
 
//initializing UserSearchService
UserSearchService userSearchService = ComponentAccessor.getComponent(UserSearchService)
     
StringBuilder output = StringBuilder.newInstance()// or new StringBuilder()
output << "<pre>\n"
 
UserSearchParams userSearchParams = new UserSearchParams.Builder()
    .allowEmptyQuery(true)
    .ignorePermissionCheck(true)
    .includeInactive(true)
    .maxResults(10000)
    .build()
 
//finding owners by passing the result of userSearchParams to loop and spit each user/owner
 
userSearchService.findUsers("", userSearchParams).each{ApplicationUser filter_owner ->
    try {
        searchRequestService.getOwnedFilters(filter_owner).each{SearchRequest filter->
            String jql = filter.getQuery().toString()
            if (jql.contains(STATUS_ONE) || jql.contains(STATUS_TWO)) {
                output << "${filter_owner.displayName}, ${filter.name}, ${filter.getPermissions().isPrivate() ? 'Private' : 'Shared'}, ${jql}\n"
 
//implement email as per team suggestion needs setup for smtp and get email for owner
            }
        }
    } catch (Exception e) {
           output << "Unable to get filters for ${filter_owner.displayName} due to ${e}"    
    }
}
 
output << "</pre>"
output.toString()
//////////////////////////////////////////////


//Script for pulling Blocked time of any Story:


import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.changehistory.ChangeHistoryManager
import com.atlassian.core.util.DateUtils
import com.atlassian.jira.issue.history.ChangeItemBean
import com.atlassian.jira.issue.Issue
import java.util.concurrent.TimeUnit
 
def issue = issue as Issue
ChangeHistoryManager changeHistoryManager = ComponentAccessor.getChangeHistoryManager()
def STATUS = "Blocked"
List<Long> blockedTime = [0L]
 
def changeFieldHistory=changeHistoryManager.getChangeItemsForField(issue, "blocked")
        .reverse();
 
if(changeFieldHistory.size()!=0){
changeFieldHistory.each { ChangeItemBean item -> def timeDiff = System.currentTimeMillis() - item.created.getTime()
 
      if (item.fromString == STATUS){
           blockedTime << -timeDiff
       }
 
      if (item.toString == STATUS){
 
           blockedTime << timeDiff
    }
                     
 }
 
}
else if(issue.getCustomFieldValue(ComponentAccessor.getCustomFieldManager().getCustomFieldObjectByName("Blocked")).toString()=='Blocked'){
    timeDiff=System.currentTimeMillis() - issue.created.getTime()
    blockedTime << timeDiff
}
 
def totalBlockedTime = Math.round((blockedTime.sum().toString().toDouble()) as Double)
 
return TimeUnit.MILLISECONDS.toDays(totalBlockedTime) + " days"





//===================================================================================================
  
//Script for Getting filters list with owner information who are using these two workflow statuses.
  
//====================================================================================================
  
import com.atlassian.jira.bc.filter.SearchRequestService
import com.atlassian.jira.issue.search.SearchRequest
import com.atlassian.jira.bc.user.search.UserSearchParams
import com.atlassian.jira.bc.user.search.UserSearchService
import com.atlassian.jira.bc.issue.search.SearchService
import com.atlassian.jira.bc.JiraServiceContext
import com.atlassian.jira.bc.JiraServiceContextImpl
import java.lang.StringBuilder
import com.atlassian.jira.user.ApplicationUser
import com.atlassian.jira.jql.parser.JqlQueryParser
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.jql.parser.JqlParseException
import com.atlassian.jira.issue.search.SearchRequest
int maxUsers=10000
SearchRequestService searchRequestService = ComponentAccessor.getComponent(SearchRequestService.class)
  
JqlQueryParser parser = ComponentAccessor.getComponent(JqlQueryParser.class)
 def searchService = ComponentAccessor.getComponent(SearchService)
UserSearchService userSearchService = ComponentAccessor.getComponent(UserSearchService)
  
StringBuilder output = StringBuilder.newInstance()
  
output << "<pre>\n"
   
UserSearchParams userSearchParams = new UserSearchParams.Builder()
    .allowEmptyQuery(true)
    .ignorePermissionCheck(true)
    .includeInactive(true)
    .maxResults(10000)
    .build()
  
userSearchService.findUsers("", userSearchParams).each{ApplicationUser filter_owner ->
  
    try {
        searchRequestService.getOwnedFilters(filter_owner).each{SearchRequest filter->
            def query = filter.query
            def jql = searchService.getJqlString(query)
           
            if (jql.contains("As Is")) {
  
               def ctx = new JiraServiceContextImpl(filter_owner)
  
                String newQueryString = jql.replace("As Is", "Active")
                              
           //   def newQuery = parser.parseQuery(newQueryString)
                def newQuery = searchService.parseQuery(filter_owner, newQueryString).query
   output << "${filter_owner.displayName}, ${filter.getId()}, ${filter.name}, ${filter.getPermissions().isPrivate() ? 'Private' : 'Shared'}<br><br> ${jql} <br><br> ${newQuery}<br><br>"
               
           if(filter.getId() == 25117){
           filter.setQuery(newQuery) 
         searchRequestService.updateFilter(ctx,filter)
                     
               }
  
            }
        }
         
    }catch (Exception ex) {
           output << "Unable to get filters for ${filter_owner.displayName} due to ${ex}<br>" 
    }
}
   
output << "</pre>"
output.toString()
//////////////////////////////////////////////////////////////////////




HIDE FIELD CONFIG

import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.fields.FieldManager;
import com.atlassian.jira.issue.fields.CustomField;
import com.atlassian.jira.issue.fields.layout.field.FieldLayoutManager;
import com.atlassian.jira.issue.fields.layout.field.FieldLayoutItem;
import com.atlassian.jira.issue.fields.layout.field.EditableFieldLayout;
import com.atlassian.jira.project.Project
import org.apache.log4j.Logger
import org.apache.log4j.Level
def logger = Logger.getLogger("testscript")
logger.setLevel(Level.DEBUG)
 
try{
 
FieldManager fieldManager = ComponentAccessor.getFieldManager();
FieldLayoutManager layoutManager = ComponentAccessor.getComponent(FieldLayoutManager)
String projKey = "CBT";
String[] usedFields = ["Acceptance Criteria","Accountable Manager","Accountable PM","Acronym for Artifact Record","Affects Version/s","Approval Needed By","Approval Reason","Approved By","Architect","Artifact","Artifact Acronym","Artifact Group","Artifact Type","Assignee","Attachment","Blocked","Blocked Notes","Business SME","Capabilities","CBT Artifact","CBT Cancellation Message","CBT Component","CBT Epic","Complexity","Component/s","Cancelled Reason","Cycle","Description","Design Authority Approval","Done to Confirmed Time","Done to Released to Customer Time","DQ Notes","DQ Status","Enter Summary","Epic Link","Epic Name","Escalation Description","Escalation Reason","First Updated Time","FIS Resources","Fix Version/s","Functional Requirement Document (FRD)","Gap ID (old)","Impact","In Epic","In Epic2","In Progress to Done Time","In Progress to In Review Time","In Progress to Validation Time","In Review to Validation Time","In Scope","Indicators","Interface(s)","Is this story an interface design story?","Issue Type","Labels","Last Comment","Latest Approval Statuses","Linked Issues","Milestone Type","Mitigation Plan","Must Have","Number of Approvers","Open to Ready Time","Out of the Box","Parent Issue Key","Parent Link","Participants","Phase","Portfolio Lead","Priority","Process Map(s)","PSID","Ready to In Progress Time","Relative Size","Report","Reporter","Resolution","Revert Back Information Text","Reviewer Comments","Ringi Sub-bucket validated?","Root Cause","Security Level","Sprint","Squad Lead","Story Points","Story Source","Story Type","Summary","System Manager","System(s)","Target end","Target start","Team","Tech SME Resources","Test Block","Test Preconditions","Time Tracking","Total Blocked Time","TPO Squad","Validation to Done","Workstream","Zephyr Teststep"] as String[]
Project nextProject = ComponentAccessor.getProjectManager().getProjectObjByKey(projKey)
def fLayouts = layoutManager.getUniqueFieldLayouts(nextProject)
for (layout in fLayouts){
 logger.debug(layout.getName())
 if (layout.getName() != null && layout.getName().equals("CBT SCRUM: Field Configuration")){
 def layoutId = layout.getId();
 EditableFieldLayout editableLayout = layoutManager.getEditableFieldLayout(layoutId);
 for (cf in layout.getFieldLayoutItems()){
 boolean isCF = fieldManager.isExistingCustomField(cf.getOrderableField().getId());
 if(isCF == true){
 
 CustomField field = fieldManager.getCustomField(cf.getOrderableField().getId());
 FieldLayoutItem item = editableLayout.getFieldLayoutItem(field);
  
    if(!usedFields.contains(cf.getOrderableField().getName())){
        if(!(item.isHidden() == true)){
            logger.debug("Hiding ${cf.getOrderableField().getName()}")
            editableLayout.hide(item);
           }
        }
      }
    }
     layoutManager.storeEditableFieldLayout(editableLayout)
  }
}}catch(Exception e)
{
    
    logger.debug(e.getMessage())
 
}
//////////////////////////////////////////////////////////////////////////////




SCRIPTS for Confluence macro

import com.atlassian.sal.api.component.ComponentLocator
import com.atlassian.confluence.pages.PageManager
import com.atlassian.confluence.pages.Page
import com.atlassian.confluence.spaces.SpaceManager
import com.atlassian.confluence.spaces.Space
import groovy.xml.MarkupBuilder
import org.jsoup.Jsoup
import org.apache.log4j.Logger
import org.apache.log4j.Level
import com.atlassian.confluence.core.DefaultSaveContext
 
def logger = Logger.getLogger("ConfluenceScript")
logger.setLevel(Level.DEBUG)
def pageManager = ComponentLocator.getComponent(PageManager)
def spaceManager = ComponentLocator.getComponent(SpaceManager)
def spacep = spaceManager.getSpace("~mua3m2w")
def pagesinfo = pageManager.getPages(spacep, true)
pagesinfo.each {
    page ->
        updatePage(page, "[ac:name='jqlQuery']", pageManager)
        updatePage(page, "[ac:name='jira_issue_filter_params']", pageManager)
}
 
 
//Get all spaces
//spaces.each { space ->
//Get pages from this space
//    def pages = pageManager.getPages(space, true)
 
/**
 * Iterate over each page from the space above and call the updatePage () function
 * and pass each page, macro name and pageManager as the parameters
 */
//   pages.each { page ->
//   updatePage(page, "[ac:name='jqlQuery']", pageManager)
//  updatePage(page, "[ac:name='jira']", pageManager)
//  }
//}
 
def updatePage(Page page, String macroName, PageManager pageManager) {
 
    def logger = Logger.getLogger("ConfluenceScript")
    logger.setLevel(Level.DEBUG)
    def page_body = page.bodyAsString
 
    /**
     * The parse(String html) method parses the input HTML into a new Document.
     * This document object can be used to traverse and get details of the html dom.
     */
    def parsedBody = Jsoup.parse(page_body)
 
    /**
     * Assuming first status macro should be changed
     * Finds the first Element that matches the supplied Evaluator, with this element as the starting context, or null if none match.
     * A selector is a chain of simple selectors, separated by combinators.
     * Selectors are case insensitive (including against elements, attributes, and attribute values).
     */
    def status_macros = parsedBody.select(macroName)
 
    status_macros.each { status_macro ->
 
        if (status_macro) {
            def jql = ""
            //if is for Macro 'jira_issue_filter_params', else if is for Macro name 'jqlQuery'
            if (macroName.equals("[ac:name='jira_issue_filter_params']")) {
                status_macro = status_macro.select("p")
                jql = status_macro.text()
                logger.debug("JQL Filter MACRO JQL :" + jql)
            } else if (macroName.equals("[ac:name='jqlQuery']")) {
                jql = status_macro.toString()
                def lines = jql.split("\\r?\\n")
                //After the tags are seperated from body the body is in second element
                jql = lines[1]
                logger.debug("JQL QUERY MACROS JQL :" + jql)
            }
            jql = jql.toLowerCase()
            //look for the string you want to replace
            if (jql.contains("done")) {
                //replace with the string you want
                String newQueryString = jql.replace("done", "cancelled")
                logger.debug("new String :" + newQueryString)
            //if is for Macro 'jira_issue_filter_params', else if is for Macro name 'jqlQuery'
                if (macroName.equals("[ac:name='jira_issue_filter_params']")) {
                    status_macro = status_macro ? status_macro.html(newQueryString) : null
                } else if (macroName.equals("[ac:name='jqlQuery']")) {
                    status_macro = status_macro ? status_macro.text(newQueryString) : null
                }
            }
        }
    }
     
    //save the page logic
    Page oldVersion = page.clone() as Page
    oldVersion.convertToHistoricalVersion()
    oldVersion.setOriginalVersion(page)
    page.setBodyAsString(parsedBody.toString())
    pageManager.saveContentEntity(page, oldVersion, DefaultSaveContext.BULK_OPERATION)
}
//////////////////////////////////////////////////////




Chnage disposition fields

//Version 2
import org.apache.log4j.Logger;
import org.apache.log4j.Level;
import com.atlassian.jira.component.ComponentAccessor;
import com.atlassian.jira.user.ApplicationUser;
import com.atlassian.jira.issue.CustomFieldManager;
import com.atlassian.jira.security.JiraAuthenticationContext
import java.util.Date;
import com.atlassian.jira.bc.issue.search.SearchService
import com.atlassian.query.Query
import com.atlassian.jira.web.bean.PagerFilter
import com.atlassian.jira.issue.search.SearchResults
  
Logger logger = Logger.getLogger("test")
logger.setLevel(Level.DEBUG)
  
ApplicationUser automationUser = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser();
CustomFieldManager customFieldManager = ComponentAccessor.getCustomFieldManager();
JiraAuthenticationContext jiraAuthenticationContext = ComponentAccessor.getJiraAuthenticationContext()
SearchService searchService = ComponentAccessor.getComponent(SearchService.class)
def changeManager = ComponentAccessor.getChangeHistoryManager()
  
def cfOverall = customFieldManager.getCustomFieldObjectsByName("CBT Disposition Overall")
def cfPhase1 = customFieldManager.getCustomFieldObjectsByName("CBT Disposition Phase1")
def cfPhase2 = customFieldManager.getCustomFieldObjectsByName("CBT Disposition Phase2")
def cfPhase3 = customFieldManager.getCustomFieldObjectsByName("CBT Disposition Phase3")
  
//Getting issues from jql
  
String jql = "project = System ";
SearchService.ParseResult parseResult = searchService.parseQuery(automationUser, jql);
if (parseResult.isValid()) {
    Query query = parseResult.getQuery()
    PagerFilter pagerFilter = new PagerFilter(2000);
    SearchResults searchResults = searchService.search(automationUser, query, pagerFilter)
    //Needs to be modified for JIRA 8 --> use getResults() instead of getIssues()
    def issues = searchResults.getResults()
    logger.debug(issues.size())
    StringBuilder result = new StringBuilder()
      
     result.append('<table border=\"1\"><tr><th>System</th><th>Previous CBT Disposition Overall</th><th>Current CBT Disposition Overall</th> <th>Updated</th>')
   .append( '<th>Previous CBT Disposition phase1</th><th>Current CBT Disposition phase1</th> <th>Updated</th>')
 .append( '<th>Previous CBT Disposition phase2</th><th>Current CBT Disposition phase2</th><th>Updated</th>')
   .append( '<th>Previous CBT Disposition phase3</th><th>Current CBT Disposition phase3</th><th>Updated</th></tr>')
      
    Date date = new Date();
    logger.debug("Current Date: " + date)
      
      
     issues.each {
        result.append("<tr><td>").append("${it.key}").append("</td><td>")
        //get custom fields value and create a updated_On field for saving changed date
        def cfOverallValue = it.getCustomFieldValue(cfOverall[0])
        def updated_On
        def cfphase1Value = it.getCustomFieldValue(cfPhase1[0])
        def updated_On_1
        def cfphase2Value = it.getCustomFieldValue(cfPhase2[0])
        def updated_On_2
        def cfphase3Value = it.getCustomFieldValue(cfPhase3[0])
        def updated_On_3
   
        //get current value for CBT Disposition Overall
        def changeItems = changeManager.getChangeItemsForField(it, "CBT Disposition Overall")
        //checking if there is something in change item for CBT disposition overall field
        if (changeItems) {
           def dateModified = new Date(changeItems.get(changeItems.size() - 1).created.getTime())
           updated_On =  dateModified.format("dd-MM-yyyy")
            //appending previous value for overall
           result.append("${changeItems.get(changeItems.size() - 1).fromString} ").append("</td><td>")
           
        } else {
            result.append("null").append("</td><td>")
        }
        //appending current value of cbt overall
        result.append("${cfOverallValue}").append("</td><td>").append("${updated_On}").append("</td><td>")
   
           
           
        //repeating the same code above for other cf's
         //for phase 1
        def changeItem1 = changeManager.getChangeItemsForField(it, "CBT Disposition Phase1")
        //checking if there is something in change item for CBT disposition overall field to append or null
        if (changeItem1) {
            def dateModified = new Date(changeItem1.get(changeItem1.size() - 1).created.getTime())
            updated_On_1 =  dateModified.format("dd-MM-yyyy")
            //appending previous value of phase 1
            result.append("${changeItem1.get(changeItem1.size() - 1).fromString}").append("</td><td>")
        } else {
            result.append("null").append("</td><td>")
        }
        //appending current value of phase1
        result.append("${cfphase1Value}").append("</td><td>").append("${updated_On_1}").append("</td><td>")
        // for phase 2
        def changeItem2 = changeManager.getChangeItemsForField(it, "CBT Disposition Phase2")
        //checking if there is something in change item for CBT disposition phase2 field to append or null
        if (changeItem2) {
            def dateModified = new Date(changeItem2.get(changeItem2.size() - 1).created.getTime())
            updated_On_2 =  dateModified.format("dd-MM-yyyy")
            //appending previous value of phase 2
          result.append("${changeItem2.get(changeItem2.size() - 1).fromString} ").append("</td><td>")
  
        } else {
            result.append("null").append("</td><td>")
        }
        //appending current value of phase2
        result.append("${cfphase2Value}").append("</td><td>").append("${updated_On_2}").append("</td><td>")
        //result.append("\n")
   
         //for phase 3
        def changeItem3 = changeManager.getChangeItemsForField(it, "CBT Disposition Phase3")
        //checking if there is something in change item for CBT disposition phase3 field to append or null
   
        if (changeItem3) {
            def dateModified = new Date(changeItem3.get(changeItem3.size() - 1).created.getTime())
            updated_On_3 =  dateModified.format("dd-MM-yyyy")
            //appending previous value of phase 3
            result.append("${changeItem3.get(changeItem3.size() - 1).fromString} ").append("</td><td>")
  
        } else {
            result.append("null").append("</td><td>")
        }
        //appending current value of phase3
         result.append("${cfphase3Value}").append("</td><td>").append("${updated_On_3}").append("</td></tr>")
   
    }
      
    result.append('</table>')
        return result
      
}
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////

Bulk edit quick filters and swimlanes

import com.atlassian.greenhopper.service.rapid.view.RapidViewService
import com.atlassian.greenhopper.web.rapid.view.RapidViewHelper
import com.atlassian.jira.component.ComponentAccessor
import com.onresolve.scriptrunner.runner.customisers.JiraAgileBean
import com.onresolve.scriptrunner.runner.customisers.WithPlugin
import com.atlassian.greenhopper.service.rapid.view.QuickFilterService
import com.atlassian.jira.user.ApplicationUser
import org.apache.log4j.Logger;
import org.apache.log4j.Level;
import com.atlassian.greenhopper.service.rapid.view.QuickFilterDao
import com.atlassian.greenhopper.service.rapid.view.SwimlaneDao
import com.atlassian.greenhopper.service.rapid.view.RapidViewAO
 
Logger logger = Logger.getLogger("TEST")
logger.setLevel(Level.DEBUG)
 
@WithPlugin("com.pyxis.greenhopper.jira")
 
@JiraAgileBean
RapidViewService rapidViewService
 
@JiraAgileBean
QuickFilterService quickFilterService
 
@JiraAgileBean
QuickFilterDao quickFilterDao
 
@JiraAgileBean
SwimlaneDao swimlaneDao
 
def currentUser = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser()
   
//ForViews: line 68 is just to test one view we are getting one view using its Long id
//def views = rapidViewService.getRapidView(currentUser,388L)
//please uncomment the below line and line which has view.each{} to get all the views and loop through to get all its quick filters and swimlane
def views = rapidViewService.getRapidViews(currentUser)
 
views.value.each {
   logger.debug(it.getId())
   //Get QuickFilters for testing on one board
    def quickFilters = quickFilterDao.getForParent(it.getId())
    //get QuickFilters if we are doing a bulk update
    //def quickFilters = quickFilterDao.getForParent(it.)
    logger.debug("---------------------------quickfilter-------------------")
//For QuickFilters: get all the quick filters for the given given Rapid view ID
quickFilters.each {
    logger.debug(it.getLongQuery()) 
    if (it.longQuery.toString().contains("mua3m2w")){
    it.setLongQuery("assignee = mua3m2e")
       logger.debug("This is new assignee is for quick filter : " +"${it.longQuery}")
       it.save()
   }
}
    logger.debug("---------------------------swimlane-------------------")
//For Swimlanes: get all the simlane for the given given Rapid view ID
    def swimlane = swimlaneDao.getForParent(it.getId())
swimlane.each{
    logger.debug(it.getLongQuery())
    if(it.getLongQuery().toString().toLowerCase().contains("medium")){
        it.setLongQuery("priority = low")
        logger.debug("This is new priority: " +"${it.longQuery}")
        it.save()  
    }
  }
}
/////////////////////////////////////////////////////////////////////////////////////////////////

Indicator

import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.CustomFieldManager
import com.atlassian.jira.issue.fields.CustomField
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.issue.MutableIssue
import com.atlassian.jira.project.version.Version
import java.util.Date
import java.text.SimpleDateFormat
import java.sql.Timestamp
import java.text.DateFormat
import groovy.time.TimeCategory
import com.atlassian.jira.issue.label.LabelManager
import com.atlassian.jira.issue.link.IssueLink
import org.apache.log4j.Logger
import org.apache.log4j.Level

def issueManager = ComponentAccessor.getIssueManager() //Get Issue Manager
CustomFieldManager customFieldManager = ComponentAccessor.getCustomFieldManager() //Get Custom Field Manager
def labelMgr = ComponentAccessor.getComponent(LabelManager) // Get Label manager
def linkManager = ComponentAccessor.issueLinkManager 	//Get issue lik Manager
def logger = Logger.getLogger("ConfluenceScript")
logger.setLevel(Level.DEBUG)

def warning1 = new StringBuilder()
def parentObj = issue.parentObject //Define parentObj as issue parent object
def epicLink = customFieldManager.getCustomFieldObjectsByName('Epic Link')[0] //Get Epic Link field object
def cftargetEnd = customFieldManager.getCustomFieldObjectsByName('Target end')[0] //Get Target end field object
def cftargetStart = customFieldManager.getCustomFieldObjectsByName('Target start')[0]//Get Target start field object
def cfaccountablePM = customFieldManager.getCustomFieldObjectsByName('Accountable PM')[0]//Get Accountable PM field object
def cfartifact = customFieldManager.getCustomFieldObjectsByName('Artifact')[0]//Get Artifacts field object
def issueTypeName = issue.issueType.name
Collection <Version> fixVersions = new ArrayList <Version> ()
fixVersions = issue.getFixVersions()
def assignee = issue.assignee
def status = issue.status.name
def epicKey = issue.getCustomFieldValue(epicLink) //Get Key value of Epic Link as string of child issue
def teamField = customFieldManager.getCustomFieldObjectsByName("Team")[0]
def team = issue.getCustomFieldValue(teamField)
def today = new Date()
def createDate = issue.getCreated()
def daysago21 = today - 21
Date targetEndVal = (Date)cftargetEnd.getValue(issue)
Date targetStartVal = (Date)cftargetStart.getValue(issue)
def sprint = customFieldManager.getCustomFieldObjectsByName("Sprint")[0]
def sprints = issue.getCustomFieldValue(sprint)
def cfBlocked = customFieldManager.getCustomFieldObjectsByName("Blocked")[0]
def cfBlockedNotes = customFieldManager.getCustomFieldObjectsByName("Blocked Notes")
def cfBlockedValue = issue.getCustomFieldValue(cfBlocked)

def cfSystems = customFieldManager.getCustomFieldObjectsByName('System(s)')[0]//Get System field object
def cfInterface = customFieldManager.getCustomFieldObjectsByName('Interface(s)')[0]//Get System field object
def cfCapabilities = customFieldManager.getCustomFieldObjectsByName('Capabilities')[0]//Get System field object
def cfProcessMaps = customFieldManager.getCustomFieldObjectsByName('Process Map(s)')[0]//Get System field object
def cfSystemsVal = issue.getCustomFieldValue(cfSystems)
def cfInterfaceVal = issue.getCustomFieldValue(cfInterface)
def cfProcessMapsVal = issue.getCustomFieldValue(cfProcessMaps)

def link = linkManager.getOutwardLinks(issue.id) //link manager object for OutwardLinks
def link1 = linkManager.getInwardLinks(issue.id) //link manager object for InwardLinks

if ((issueTypeName != "Workstream") && (issueTypeName != "Artifact")) {
//For All these items below: project = CBT and then exclude issuetype != Workstream, Artifact and Status = Open

	//--Status Indicator: Late Start (font color = orange)--
	//status not in (Done, Cancelled) AND "Target start" < startOfDay() AND status = ready AND "Target end" >= startOfDay()
	//  "Target start" < startOfDay() AND status = ready
	if(status == "Ready" && targetStartVal?.before(today)) 
	{
    	warning1 <<= '<font color="orange"><i>STATUS: Late Start</i></br></font>'
	}


	//--Status Indicator: Missed Finish (font color = red)--
	//status not in (Done, Cancelled) AND "Target end" < startOfDay()
	if (targetEndVal) {
		if(status != "Done" && status != "Cancelled" && targetEndVal?.before(today-1)) 
		{
    		warning1 <<= '<font color="red"><i>STATUS: Missed Finish</i></br></font>'
		}
	}


	//--QA Indicator: Open Status (font color = maroon)--
	//status in (Open) & CreateDate > 21days
	def daysOpen = today - createDate
		//log.error("today: " + today)
		//log.error("daysago21: " + daysago21)
		//log.error("daysOpen: " + daysOpen.days)

	if	(status == "Open" && createDate?.before(daysago21))
	{
    	warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Open > 21days</i></br></font>'
	}
	

    //--QA Indicator: Missing Start (font color = maroon)--
    //status not in (Cancelled) AND "Target start" = EMPTY
    if(status != "Cancelled")
    {
        if (!targetStartVal)
        {
            warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Target Start Date</i></br></font>'
        }
        
        //--QA Indicator: Missing End Date (font color = maroon)--
    	//status not in (Cancelled) AND "Target end" = EMPTY
         if (!targetEndVal)
        {
            warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Target End Date</i></br></font>'
        }
        
        //--QA Indicator: Missing PM (font color = maroon)--
    	//status not in (Cancelled) AND "Accountable PM" is EMPTY
        if (!cfaccountablePM.getValue(issue))
        {
            warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Accountable PM</i></br></font>'
        }
        
        //--QA Indicator: Missing Team (font color = maroon)--
    	//status not in (Cancelled) AND issuetype in (Story, Milestone) AND Team is EMPTY
        if (issueTypeName == "Story"||issueTypeName == "Milestone")
            if (!team)
        {
            warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Team</i></br></font>'
        }
    }

    //--QA Indicator: Missing Assignee (font color = maroon)--
    //status not in (Cancelled) AND Assignee is EMPTY
    if(status != "Cancelled" && status != "Done")
    {
        if (!assignee)
        {
            warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Assignee</i></br></font>'
        }
    }
  
   
    //--QA Indicator: Missing FixVersion (font color = maroon)--
    //status not in (Cancelled) AND fixVersion is EMPTY
    if(issueTypeName != "Sub-task" && issueTypeName != "Escaltion" && status != "Cancelled")
    {
        if (!fixVersions)
        {
            warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Fix Versions</i></br></font>'
        }
    }
    
    //QA Indicator: Missing Artifact (font color = maroon)--
	//status not in (Cancelled) AND issuetype in (Story, Milestone, Sub-Task) AND Artifact is EMPTY
    if(status != "Cancelled")
    {	if (issueTypeName == "Story"||issueTypeName == "Milestone"||issueTypeName == "Sub-task")
     	{
            if (!cfartifact.getValue(issue)) 
            {
      			warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Artifact</i></br></font>'          
            }
        }
    }
    
   // Missign Sprint - if sprint is missing on the Story and Target End date is after May 2020
    if(status != "Cancelled" && issueTypeName == "Story" && status != "Open")
    {
        /*if (!sprints)
        {
            warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Sprint</i></br></font>'
        }*/
         Date staticDate = new Date().parse("MMM/yyyy", "May/2020")
         logger.debug("staticDate " + staticDate)
        
        if(targetEndVal?.before(staticDate))
        {
            logger.debug("targetEndVal " + targetEndVal)
            //Don't show missing Sprint
            warning1 <<= ''  
        }
        else if (!sprints)
        {
            warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Sprint</i></br></font>'
        }
        
        
    }
    
    // Missing Blocked Notes - if Blocked = Yes and Blocked notes are empty
    // project = "CBT Execution" and issuetype  in (story, Sub-task, Milestone, Epic) and blocked = Blocked and "Blocked Notes" is EMPTY
    if(status != "Cancelled" && status != "Open")
    {	
        
        if (issueTypeName == "Story" || issueTypeName == "Epic" || issueTypeName == "Milestone" || issueTypeName == "Sub-task")
     	{
            //logger.debug(cfBlockedValue)
        	def cfBlockedNotesValue = issue.getCustomFieldValue(cfBlockedNotes[0])
       
            if (cfBlockedValue.toString() == "Blocked")
            {
                             
            	if (!cfBlockedNotesValue) 
            	{
      				warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Blocked Notes</i></br></font>'          
            	}
                
                //project = cbt AND status not in (Cancelled, Open) AND Blocked = Blocked  and issueLinkType != "is blocked by"
                //Proposed indicator language:
				//DATA QUALITY: Missing “is blocked by” link(s), Applied to stories, subtasks, epics, and milestones issue types. 
                def islinkfound = false
                link1.each 
        		{ 
                   	//logger.debug('linktype '+ it.issueLinkType.id.toString())
                     if(it.issueLinkType.id.toString() == '10000') //link id for "is blocked by" id = 10000
                    {
                       
                        islinkfound = true
                    }
                    
        		}
                
                //logger.debug('linkFound ' + islinkfound)
                if (!islinkfound)
                {
                    
                   warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing “is blocked by” link(s)</i></br></font>'   
                }
            }
        }
        
        // Interface Story Missing a System - Interface story missing a system
    	// project = cbt AND issuetype = story and status not in (Cancelled, open) AND "Interface(s)" is not EMPTY AND "System(s)" is EMPTY
        if (issueTypeName == "Story")
        {
            //logger.debug('interface' + cfInterfaceVal)
            //logger.debug('system' + cfSystemsVal)
            if(cfInterfaceVal && !cfSystemsVal)
            {
                warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Interface Story Missing a System</i></br></font>'
            }
        }
    }

	 
        
    // -Missing Systems: a) Based on Label: If the following labels are present and system field is missing
	//ITPD.Deliverable.FDD.System ,ITPD.Deliverable.FDD.System,ITPD.Deliverable.DDD.IntegrationDesign Except when there is the label Design.Testing.Only
	//JQL : project = cbt AND status not in (Cancelled, open) AND issuetype = Story AND ("System(s)" is EMPTY AND labels in (ITPD.Deliverable.FDD.System, ITPD.Deliverable.DDD.System, ITPD.Deliverable.DDD.IntegrationDesign)
    //b) Based on having System(s) related to Work link : If system link is present in issue link but if system field is missing 
	//JQL : project = cbt AND status != Cancelled AND (issueLinkType = "System(s) related to Work" AND "System(s)" is EMPTY”
	if(status != "Cancelled" && (issueTypeName == "Story"))
    {
        
        //logger.debug('link1 ' + link.size())
        def valueS = link.findAll{(it.issueLinkType.id.toString() == '11900')}
        valueS.find { 
       
    		//logger.debug('value ' + value)
            if(!cfSystemsVal)
            {
                //logger.debug('system ' + cfSystemsVal)
                warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing System based on link</i></br></font>' 
            }
  		}
        
       if(issue.getLabels()?.any{(it.label == "ITPD.Deliverable.FDD.System") || (it.label == "ITPD.Deliverable.DDD.System") || (it.label == "ITPD.Deliverable.DDD.IntegrationDesign")})
        {
             if(!cfSystemsVal)
            {
                //logger.debug('system ' + cfSystemsVal)
                warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing System based on Label</i></br></font>' 
            }
        }
        
         //logger.debug('interface ' + cfInterfaceVal)
        //logger.debug('link2 ' + link.issueLinkType.id.toString())
        def valueI = link.findAll{(it.issueLinkType.id.toString() == '12300')} // Interface(s) related to Work link id = 12300
        valueI.find 
        { 
           	//logger.debug('value ' + value)
            if(!cfInterfaceVal)
            {
                warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Interface based on Link</i></br></font>' 
            }
  		}
        
        if(issue.getLabels()?.any{(it.label == "ITPD.Deliverable.DDD.IntegrationDesign")})
        {
            if(!cfInterfaceVal)
            {
                warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Interface based on Label</i></br></font>' 
            }
        }
        
        //logger.debug('Process Maps ' + cfProcessMapsVal)
        //logger.debug('link2 ' + link.issueLinkType.id.toString())
        def valueP = link.findAll{(it.issueLinkType.id.toString() == '12301')} // MAP(s) related to Work = 12301
        valueP.find 
        { 
           	
            if(!cfProcessMapsVal)
            {
                warning1 <<= '<font color="#9c0396"><i>DATA QUALITY: Missing Process Maps</i></br></font>' 
            }
  		}
    }
    
    
    
    //--Hierarchy Indicator: Orphan Story (font color = hot pink)[--]
    //issuetype in (Story, Milestone) AND "Epic Link" is EMPTY
    if (status != "Cancelled" && !epicKey && (issueTypeName == "Story" || issueTypeName == "Milestone"))
    {
        warning1 <<= '<font color="#FF69B4"><i>HIERARCHY: Orphaned</i></br></font>'

    }


    //--Hierarchy Indicator: Orphan Sub-task (font color = hot pink)[--]
    //issuetype = Sub-task AND "parent" is EMPTY
    if (status != "Cancelled" && issueTypeName == "Sub-task" && !parentObj)
    {
        logger.debug("parent" + parentObj)
        warning1 <<= '<font color="#FF69B4"><i>HIERARCHY: Orphaned Sub-task</i></br></font>'

    }
    
    //--Hierarchy Indicator: Wrong parent type (font color = hot pink)[--]
    //issuetype = Sub-task AND "has parent" and parent issue type = Epic
     if (status != "Cancelled" && issueTypeName == "Sub-task" && parentObj?.issueType?.name != 'Story')
    {
        warning1 <<= '<font color="#FF69B4"><i>HIERARCHY: Wrong parent type </i></br></font>'

    }
         


    //--Date Warning: Child Start Date Before Parent (font color = black)--
    //Check for Child Start date is BEFORE Parent Start date
    //For Story Issue Type
    if (issueTypeName == "Story" && epicKey) {
        def epicObj = issueManager.getIssueObject(epicKey.toString()) //Return issue object from EpicLink Key
        def pTargetStart = epicObj?.getCustomFieldValue(cftargetStart) as Date //Get parent Target Start
        def cTargetStart = issue.getCustomFieldValue(cftargetStart) as Date//Get Target Start of child issue
            //log.error(epickey.toString())
            //log.error('cTargetStart: ' + cTargetStart)
            //log.error('pTargetStart: ' + pTargetStart)

        if (cTargetStart) {
            if (pTargetStart) {
                if (cTargetStart.before(pTargetStart)) {
                    warning1 <<= '<font color="black"><i>DATE: Target start date before parent Epic Target start date</i></font></br>'
                } else {
                    warning1 <<= ''
                }
            } else {
                warning1 <<= '<font color="black"><i>DATE: Parent Target start date is null</i></font></br>'
            } 
        } else {
            warning1 <<= ''
        }   
    } else if (issueTypeName == "Sub-task") {
        if (parentObj) {
            def pTargetStart = parentObj?.getCustomFieldValue(cftargetStart) as Date //Get parent Target Start
            def cTargetStart = issue.getCustomFieldValue(cftargetStart) as Date//Get Target Start of child issue
            if (cTargetStart) {
                if (pTargetStart) {
                    if (cTargetStart.before(pTargetStart)) {
                        warning1 <<= '<font color="black"><i>DATE: Target start date before parent Story Target start date</i></font></br>'

                    } else {  
                    warning1 <<= ''
                    }
            } else {
                warning1 <<= '<font color="black"><i>DATE: Parent Target start date is null</i></font></br>'
            } 
        } else {
            warning1 <<= ''
        }    
        }
    }



    //--Date Warning: Child End Date After Parent (font color = black)--
    //Child End date is AFTER Parent End Date
    if (issueTypeName == "Story" && epicKey) {
        def epicObj = issueManager.getIssueObject(epicKey.toString()) //Return issue object from EpicLink Key
        def pTargetEnd = epicObj?.getCustomFieldValue(cftargetEnd) as Date //Get Key value of Epic Link as string
        def cTargetEnd = issue.getCustomFieldValue(cftargetEnd) as Date//Get Target End of child issue
            //log.error(epickey.toString())
            //log.error('cTargetEnd: ' + cTargetEnd)
            //log.error('pTargetEnd: ' + pTargetEnd)

        if (cTargetEnd) {
            if (pTargetEnd) {
                if (cTargetEnd.after(pTargetEnd)) {
                    warning1 <<= '<font color="black"><i>DATE: Target end date after parent Epic Target end date</i></font></br>'
                } else {
                    warning1 <<= ''
                }
            } else {
                //warning1 <<= '<font color="black"><i>DATE: Parent Target end is null</i></font></br>'
            } 
        } else {
            warning1 <<= ''
        }    
    } else if (issueTypeName == "Sub-task") {
        if (parentObj) {
            def pTargetEnd = parentObj?.getCustomFieldValue(cftargetEnd) as Date //Get parent Target End
            def cTargetEnd = issue.getCustomFieldValue(cftargetEnd) as Date//Get Target End of child issue
            if (cTargetEnd) {
                if (pTargetEnd) {
                    if (cTargetEnd.after(pTargetEnd)) {
                        warning1 <<= '<font color="black"><i>DATE: Target end date after parent Story Target end date</i></font></br>'

                    } else {  
                    warning1 <<= ''
                    }
            } else {
                //warning1 <<= '<font color="black"><i>DATE: Parent Target start is null</i></font></br>'
            } 
        } else {
            warning1 <<= ''
        }    
        }
    }
    
    if (!warning1) 
    {
       warning1 = 'None'
    }
    
return warning1
}
///////////////////////////////////////////////////////////////////////////////////////////////////////////////

Related design

import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.link.IssueLink
import org.apache.log4j.Logger
import org.apache.log4j.Level
def logger = Logger.getLogger("Related Development")
logger.setLevel(Level.DEBUG)
//return //* this return can be enabled for disabling this scripted field
def customFieldManager = ComponentAccessor.getCustomFieldManager()
def aAcronym = customFieldManager.getCustomFieldObjectsByName("Artifact Acronym")[0] 
def linkManager = ComponentAccessor.issueLinkManager
StringBuilder sb = new StringBuilder() //Define string builder
def counter= 0
def value = linkManager.getInwardLinks(issue.id).findAll{(it.issueLinkType.id.toString() == '11900' || it.issueLinkType.id.toString() == '12209' || it.issueLinkType.id.toString() == '12300') && !(it.getSourceObject()?.status?.name == 'Cancelled')}
value.find { 
    Set issueLabels = it.getSourceObject().getLabels()
	boolean containsLabel = true
       
    def acronym = it.getSourceObject().getCustomFieldValue(aAcronym)
    
    if(acronym.toString().contains("FDD"))
    {
               
        if(!issueLabels.toString().contains("ITPD.Deliverable.FDD.FR") && !issueLabels.toString().contains("CBT.Gap")) 
        {
            sb.append('<a href = "https://prod-1-jira.mufgamericas.com/browse/')
              .append(it.getSourceObject().getKey().toString()).append( '">')
              .append('<img src = "https://prod-1-jira.mufgamericas.com/images/icons/issuetypes/story.svg"> ')
              .append(it.getSourceObject().getKey().toString()).append( ' - ')
              .append( it.getSourceObject().getSummary().toString())
              .append( ' | <span class=" jira-issue-status-lozenge aui-lozenge jira-issue-status-lozenge-yellow jira-issue-status-lozenge-indeterminate aui-lozenge-subtle jira-issue-status-lozenge-max-width-short" data-tooltip="<span class="jira-issue-status-tooltip-title">' )
              .append( it.getSourceObject().getStatus().name.toString()).append( '</span></a></br>')
             if (++counter >= 50) {
      return true
     }
        }
    	  
    }
    
              //logger.debug(acronym)
	// Design is any of the work item where Artifact Acronym = FDD, DDD, API, DDDAO or WBK
    if(acronym.toString().contains("DDD")|| acronym.toString().contains("API")|| acronym.toString().contains("DDDAO")|| acronym.toString().contains("WBK"))
    {
            
        	//logger.debug(acronym)
            sb.append('<a href = "https://prod-1-jira.mufgamericas.com/browse/')
              .append(it.getSourceObject().getKey().toString()).append( '">')
              .append('<img src = "https://prod-1-jira.mufgamericas.com/images/icons/issuetypes/story.svg"> ')
              .append(it.getSourceObject().getKey().toString()).append( ' - ')
              .append( it.getSourceObject().getSummary().toString())
              .append( ' | <span class=" jira-issue-status-lozenge aui-lozenge jira-issue-status-lozenge-yellow jira-issue-status-lozenge-indeterminate aui-lozenge-subtle jira-issue-status-lozenge-max-width-short" data-tooltip="<span class="jira-issue-status-tooltip-title">' )
              .append( it.getSourceObject().getStatus().name.toString()).append( '</span></a></br>')
        
        if (++counter >= 50) {
      return true
     }

    }
    return false    
} 

//if (sb.length() == 0 ) { //Test ff string builder is still empty
//	sb.append( "There are no related design.")
//}
return sb.toString()



/////////////////////////////////////////////////////////////////////////////////////////////////

LISTENER


import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.IssueManager
import com.atlassian.jira.issue.fields.CustomField;
import com.atlassian.jira.issue.util.DefaultIssueChangeHolder
import com.atlassian.jira.issue.ModifiedValue
import com.atlassian.jira.issue.link.IssueLink
import com.atlassian.jira.issue.Issue
import com.atlassian.jira.event.issue.link.IssueLinkCreatedEvent
import com.atlassian.jira.event.issue.link.IssueLinkDeletedEvent
import com.atlassian.jira.event.issue.IssueEvent
import java.text.SimpleDateFormat;
import com.atlassian.jira.issue.CustomFieldManager
import com.atlassian.jira.issue.search.SearchProvider
import com.atlassian.jira.bc.issue.search.SearchService
import com.atlassian.jira.web.bean.PagerFilter
import com.atlassian.query.Query
import com.atlassian.jira.issue.search.SearchResults
import com.atlassian.jira.issue.search.SearchQuery
import com.atlassian.jira.issue.customfields.option.Option;
import com.atlassian.jira.issue.link.IssueLinkManager
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.link.IssueLinkTypeManager

///import com.atlassian.jira.issue.customfields.option.Options;
import org.apache.log4j.Logger
import org.apache.log4j.Level


List<String> systemsKey = new ArrayList<String>()
List<String> epicKey = new ArrayList<String>()
List<Option> options = new ArrayList<Option>()


def issueManager = ComponentAccessor.getComponentOfType(IssueManager);
def logger = Logger.getLogger("ConfluenceScript")
def linkMgr = ComponentAccessor.getIssueLinkManager()
def customFieldManager = ComponentAccessor.getCustomFieldManager()

def userManager = ComponentAccessor.getUserManager();
def optionsManager = ComponentAccessor.getOptionsManager()
def searchProvider = ComponentAccessor.getComponentOfType(SearchProvider.class);
def searchService = ComponentAccessor.getComponentOfType(SearchService.class);
def appUser = userManager.getUserByName("Automation");
def eventTypeManager = ComponentAccessor.getEventTypeManager()
def issueLinkTypeManager = ComponentAccessor.getComponent(IssueLinkTypeManager)

/**Getting the System field, Epic link, Test Environment, PSID and Alm Application System Field**/
def epicLinkFiled = customFieldManager.getCustomFieldObject("customfield_10101");
def testEnvironment = customFieldManager.getCustomFieldObject("customfield_16139");
def systesmcField = customFieldManager.getCustomFieldObject("customfield_15402");
def psIdField = customFieldManager.getCustomFieldObject('customfield_15603')
def almApplicationSystem = customFieldManager.getCustomFieldObject('customfield_20200')
def winnerEpic
    
//If issue event is of type issuelink created
//logger.debug(event)
if([IssueLinkCreatedEvent].contains(event.getClass()) || [IssueLinkDeletedEvent].contains(event.getClass())){

    def events
    	if([IssueLinkCreatedEvent].contains(event.getClass())){		
        	events = event as IssueLinkCreatedEvent
        }
       else {
           if([IssueLinkDeletedEvent].contains(event.getClass())){
              events = event as IssueLinkDeletedEvent
           }
       }
       
       def issueKey =  events.getIssueLink().getSourceObject()//Getting id of the sourceObject
	   def issue = issueManager.getIssueByCurrentKey(issueKey.toString())
       def reporter =issue.getReporter().getUsername()
       def issueType = issue.getIssueType().getName()//Getting the issuetype of Source Object
       //logger.debug(issueKey.toString()+issueType)
       
       if(issueType == 'Test' && reporter == 'svc_alm'){	//If new Test issue is added to the Test case issue, it will read the epic link and Test env. from the test case issue and update it on test issue

                    def testIssuesLink =  linkMgr.getOutwardLinks(issue.id).findAll{ it.issueLinkType.id == 10003}  
           			if(testIssuesLink.size()>0){
                            def testCaseIssue = testIssuesLink[0].getDestinationObject()//Getting the test case issue 
                            def testCaseIssueEpicLinkFiledValue = testCaseIssue.getCustomFieldValue(epicLinkFiled)//Getting teh epic link value from test case issue
                            def testCaseTestEnvironment = testCaseIssue.getCustomFieldValue(testEnvironment)//Getting the test env. field value from the test case issue
                            //Updating epic link and test env field value on test issue.
                            if(testIssuesLink[0].getSourceObject().getCustomFieldValue(epicLinkFiled) != testCaseIssueEpicLinkFiledValue){
                                epicLinkFiled.updateValue(null, testIssuesLink[0].getSourceObject(), new ModifiedValue(testIssuesLink[0].getSourceObject().getCustomFieldValue(epicLinkFiled), testCaseIssueEpicLinkFiledValue), new DefaultIssueChangeHolder())//Updating the epic link of test.
                            }
                            testEnvironment.updateValue(null, testIssuesLink[0].getSourceObject(), new ModifiedValue(testCaseIssue.getCustomFieldValue(testEnvironment), testCaseTestEnvironment), new DefaultIssueChangeHolder())
                    }

       }
      else if(issueType == 'Bug' && reporter == 'svc_alm'){ //If new Bug issue is added to the Test issue, it will read the epic link from the test issue and update the epic link, If there is multiple test linked to Bug, then epic logic will be used.
                    
          if([IssueLinkCreatedEvent].contains(event.getClass()) || [IssueLinkDeletedEvent].contains(event.getClass())){
                    def bugIssuesLink = linkMgr.getOutwardLinks(issue.id).findAll{ it.issueLinkType.id == 10000}//Get the test issues 
                    //logger.debug("Test issue link"+bugIssuesLink.size())
                    if(bugIssuesLink.size() >0){
                      
                        def testIssueEpicLinkFiledValue 
                        bugIssuesLink.each{ //Iterate for each test case to get the primary epic using the Epic logic
                            def testIssue = it.getDestinationObject()//Get the test issue
                            def testReporter = testIssue.getReporter().getUsername()//Getting the reporter
                            //logger.debug(testReporter)
                            if(testReporter == 'svc_alm'){//Checking if Svc_alm is the reporter or not on Test issue
                                //logger.debug(testIssue)
                                
                                testIssueEpicLinkFiledValue = testIssue.getCustomFieldValue(epicLinkFiled)//Reading the test issue epic link field
                                if(testIssueEpicLinkFiledValue!=null){
                                    def primaryEpics = linkMgr.getOutwardLinks(issueManager.getIssueObject(testIssueEpicLinkFiledValue.toString()).id).findAll{ it.issueLinkType.id == 10307}//Getting the  Primary Epic link to test epic
                                    if(primaryEpics.size()!= 0){
                                        def primaryEpic = primaryEpics[0].getDestinationObject()//Getting the primary epic
                                        def primaryEpicKey = primaryEpic.getKey()//Getting primary epic key
                                        epicKey.add(primaryEpicKey)//Adding primary key to the epicKey List
                                    }
                                }
                            }
                        }
                        //logger.debug(epicKey)
                        if(epicKey.size() ==1){//If epic key size is equal to 1.
                            //Update the epic linkfield value on Bug issue  same as epic link value on Test case
                            if(issue.getCustomFieldValue(epicLinkFiled)!= issueManager.getIssueObject(testIssueEpicLinkFiledValue.toString())){//Update epic link only if there is change in epic link
                                epicLinkFiled.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(epicLinkFiled), issueManager.getIssueObject(testIssueEpicLinkFiledValue.toString())), new DefaultIssueChangeHolder())
                            }
                        } 
                        else if(epicKey.size()== 0){//If there is no epic found linked to test case or if there is no test case linked
                            if(issue.getCustomFieldValue(epicLinkFiled)!= null)//Update epic link only if there is change in epic link
                                epicLinkFiled.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(epicLinkFiled), null), new DefaultIssueChangeHolder())
                        }
                        else{	
                                
                                winnerEpic = getEpic(epicKey,issueManager, customFieldManager,linkMgr) //Using epic logic to calculate the epic for bug linked with multiple test
                                if(issue.getCustomFieldValue(epicLinkFiled) != winnerEpic){//Update epic link only if there is change in epic link
                                    epicLinkFiled.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(epicLinkFiled), winnerEpic), new DefaultIssueChangeHolder())//Update the epic link on the bug using the epic provided the GetEpic function
                                }
                          }
                        
                    }
                else{
                         //Epic link update on Bug if there is no test is linked to bug
                         if(issue.getCustomFieldValue(epicLinkFiled)!= null){
                            epicLinkFiled.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(epicLinkFiled), null), new DefaultIssueChangeHolder())
                         }
                }
          }
      }
}
else{
    //If the event is of type issueevent 
    if([IssueEvent].contains(event.getClass())){

         def events = event as IssueEvent//Intitalizing events as issue Event
       
         def issue = events.issue as Issue // Getting the current issue
         def issueType = issue.getIssueType()?.name//Getting the current issue type
         def reporter =issue.getReporter().getUsername()
        
        
         if(((issueType == 'Test Case' || issueType == 'Bug') && reporter == 'svc_alm' )){//Checking if reporter is svc_alm for test case and bug issue
                def eventTypeName = eventTypeManager.getEventType(events.eventTypeId).getName()//Getting the eventtype name
                def almApplicationSystemValue = null
                def allOptions = ComponentAccessor.getOptionsManager()
                        .getOptions(psIdField.getConfigurationSchemes().listIterator().next().getOneAndOnlyConfig());//Reading all the options of PSID field
                def almApplicationSystemChange =  events?.getChangeLog()?.getRelated("ChildChangeItem")?.find {it.field == "ALM Application/System"}//Reading the change in Alm Application System field
                def summaryChange = events?.getChangeLog()?.getRelated("ChildChangeItem")?.find {it.field == "summary"}//Reading the change in Summary field
                def testEnvironmentChange =  events?.getChangeLog()?.getRelated("ChildChangeItem")?.find {it.field == "Test Environment"}
             
				log.warn(almApplicationSystemChange)
                if(eventTypeName == 'Issue Created' &&  issueType == 'Test Case'){//If even is issue created and type is Test case

                        def summary = issue.getSummary()//Reading the summary
                        def summarySplit = summary.split('_')//Split the summmary by '_'
                        if(summarySplit.size() >0){
                            def epicKeySummary = summarySplit[0]//Read the first element of the summary
                            //logger.debug(epicKeySummary)
                            def epic = issueManager.getIssueByCurrentKey(epicKeySummary)//Get the epic using the epicKey extracted from Summary
                            if(epic!=null){
                                def testEpics = linkMgr.getInwardLinks(epic.id).findAll{it -> it.issueLinkType.id == 10307}//Reading the testing epic linked to primary epic.
                                if(testEpics.size() != 0){//Proceed further only if there is test epic linked to the primary epic
                                    def testEpic
                                    testEpics.each{ testEpic = it.getSourceObject().getKey()}
                                    epicLinkFiled.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(epicLinkFiled), issueManager.getIssueByCurrentKey(testEpic.toString())), new DefaultIssueChangeHolder())
                                }
                                
                            }
                        }
                    //Updating the PSID and System field by reading the value from the Alm Application System Field
                    almApplicationSystemValue = issue.getCustomFieldValue(almApplicationSystem)
                    if(almApplicationSystemValue != ''){
                         def almApplicationSystemValueList = almApplicationSystemValue.toString().replaceAll("\\s","").split(',') //Split the Alm Application System field value by ',' and replace extra space
                         updatePsIDSystem(almApplicationSystemValueList,customFieldManager,psIdField,DefaultIssueChangeHolder,ModifiedValue,issue)
                    }
                }
                else if(summaryChange &&  issueType == 'Test Case'){//If summary is updated on test case issue, following code will be executed
						
                        //Extracting the epic key from the summary
                        def summary = issue.getSummary()//Get the Summary from the issue
                        def summarySplit = summary.split('_')//Split the summmary by '_'
                        if(summarySplit.size() >0){
                            def epicKeySummary = summarySplit[0]//Read the first element of the summary
                            def epic = issueManager.getIssueByCurrentKey(epicKeySummary.toString())//Get the epic using the epicKey extracted from Summary
                            //logger.debug("Summary"+epic)
                            if(epic != null){//If Epic is not null
                                def testEpics = linkMgr.getInwardLinks(epic.id).findAll{it -> it.issueLinkType.id == 10307}//Reading the testing epic linked to primary epic.
                                //logger.debug("Test Epic"+testEpics)
                                if(testEpics.size() != 0){//If TestEpic is linked to Primary Epic
                                    def testEpic
                                    testEpics.each{ testEpic = it.getSourceObject().getKey()}
                                    //Updating the epic link on test case 
                                    if(issue.getCustomFieldValue(epicLinkFiled) !=  issueManager.getIssueByCurrentKey(testEpic.toString())){//Update the epic link only if there is change in epic link on the test case issue
                                    	epicLinkFiled.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(epicLinkFiled), issueManager.getIssueByCurrentKey(testEpic.toString())), new DefaultIssueChangeHolder())
                                    }
                                    //Geting Test issue linked to test case
                                    def testIssuesLink = linkMgr.getInwardLinks(issue.id).findAll{ it.issueLinkType.id == 10003}
                                    testIssuesLink.each{
                                           
                                            epicKey = []
                                            //logger.debug("Summary test issue"+it.getSourceObject())
                                            def testReporter = it.getSourceObject().getReporter().getUsername()//Getting the reporter onthe test issue
                                            if(testReporter == 'svc_alm'){//Checking if reporter assigned on the test issue is svc_alm
                                                    if(it.getSourceObject().getCustomFieldValue(epicLinkFiled) !=  issueManager.getIssueByCurrentKey(testEpic.toString())){
                                                   	 epicLinkFiled.updateValue(null, it.getSourceObject(), new ModifiedValue(it.getSourceObject().getCustomFieldValue(epicLinkFiled), issueManager.getIssueByCurrentKey(testEpic.toString())), new DefaultIssueChangeHolder())
                                                    }
                                                    def bugIssuesLink = linkMgr.getInwardLinks(it.sourceId).findAll{ it.issueLinkType.id == 10000}//Fetching bug issue
                                                    //Iterating over bug issue link
                                                    bugIssuesLink.each{  
                                                         
                                                         def bugIssue = it.getSourceObject()//Getting the bug issue
                                                         def bugReporter = bugIssue.getReporter().getUsername()//Getting the reporter on bug issue
                                                         if(bugReporter == 'svc_alm'){//Update epic link on bug only if reporter=svc_alm
                                                             def testIssueLinks = linkMgr.getOutwardLinks(bugIssue.id).findAll{ it.issueLinkType.id == 10000}//Getting the test issue linked to bug
                                                             def testIssueEpicLinkFiledValue 

                                                             //Updating the epic link on bug issue
                                                             //logger.debug("Summary Bug issue"+bugIssue)
                                                             testIssueLinks.each{
                                                                
                                                                def testIssue = it.getDestinationObject()//reading the testIssue
                                                                testReporter = it.getSourceObject().getReporter().getUsername()
                                                                 if(testReporter == 'svc_alm'){//Checking reporter on test issue
                                                                    //logger.debug("Summary test issue"+testIssue)
                                                                    testIssueEpicLinkFiledValue = testIssue.getCustomFieldValue(epicLinkFiled)//Reading the testing epic link on test issue
                                                                     if(testIssueEpicLinkFiledValue != null){
                                                                            def primaryEpics = linkMgr.getOutwardLinks(issueManager.getIssueObject(testIssueEpicLinkFiledValue.toString()).id).findAll{ it.issueLinkType.id == 10307}//Reading the Primary Epic linked to testing epic
                                                                            if(primaryEpics.size() != 0){//If pRimary epic is linked to testing epic, it will proceed further
                                                                                def primaryEpic = primaryEpics[0].getDestinationObject()
                                                                                def primaryEpicKey = primaryEpic.getKey()
                                                                                epicKey.add(primaryEpicKey)//Adding the primary ket to epicKey List. 
                                                                            }
                                                                     }
                                                                  }
                                                              }
                                                              //logger.debug("Summary epicKey"+epicKey)
                                                              //If only one Test issue linked to Bug and the epic is different from the current epic linked to bug
                                                              if(epicKey.size() ==1 &&  (bugIssue.getCustomFieldValue(epicLinkFiled) !=  issueManager.getIssueObject(testIssueEpicLinkFiledValue.toString()))){
                                                                    //Update the epic linkfield value on Bug issue
                                                                  
                                                                    epicLinkFiled.updateValue(null, bugIssue, new ModifiedValue(bugIssue.getCustomFieldValue(epicLinkFiled), issueManager.getIssueObject(testIssueEpicLinkFiledValue.toString())), new DefaultIssueChangeHolder())
                                                              } 
                                                              //If no test issue linked to bug and Epic is linked to bug
                                                              else if(epicKey.size()== 0 && (bugIssue.getCustomFieldValue(epicLinkFiled) != null)){
                                                                    //Clear the epic link field
                                                                    epicLinkFiled.updateValue(null, bugIssue, new ModifiedValue(bugIssue.getCustomFieldValue(epicLinkFiled), null), new DefaultIssueChangeHolder())
                                                              }
                                                              else{//If there is multiple Test issue linked to bug
                                                                        winnerEpic = getEpic(epicKey,issueManager, customFieldManager,linkMgr) //Call get epic function
                                                                        if(bugIssue.getCustomFieldValue(epicLinkFiled) != winnerEpic){//if the epic is currently linked to bug, is different from winner epic, then update the bug epic link
                                                                         epicLinkFiled.updateValue(null, bugIssue, new ModifiedValue(bugIssue.getCustomFieldValue(epicLinkFiled), winnerEpic), new DefaultIssueChangeHolder())
                                                                        }
                                                              }
                                                       }

                                                    }
                                            }
                                    }
                                }
                                else{
                                    clearEpic(issue, epicLinkFiled)//Clear epic links on Test case , Test and Update the epic links on bug
                                }
                            }else{
                                clearEpic(issue, epicLinkFiled)//Clear epic links on Test case , Test and Update the epic links on bug
                            }
                        }else{
                            clearEpic(issue, epicLinkFiled)//Clear epic links on Test case , Test and Update the epic links on bug
                        }

                }
               else if(testEnvironmentChange &&   issueType == 'Test Case' ){
                   //Reading the test issue linked to Test case
                   def testIssuesLink = linkMgr.getInwardLinks(issue.id).findAll{ it.issueLinkType.id == 10003}
                   testIssuesLink.each{
                      def testIssue = it.getSourceObject()
                      def testReporter = testIssue.getReporter().getUsername()//Getting the reporter onthe test issue
                      if(testReporter == 'svc_alm'){
                          def fildeConfig = (List)testEnvironment.getConfigurationSchemes()
                          def testEnvAllOptions = ComponentAccessor.getOptionsManager().getOptions(testEnvironment.getConfigurationSchemes()[1].getOneAndOnlyConfig())
                          def optionToSelect = (testEnvAllOptions.find{//finding the Option to select for test env field
                            it.value == testEnvironmentChange.newstring
                            })
                          testEnvironment.updateValue(null, testIssue, new ModifiedValue(testIssue.getCustomFieldValue(testEnvironment), optionToSelect), new DefaultIssueChangeHolder())
                          
                      }
                       
                   }
                   
                 
               }
               else if(almApplicationSystemChange){
                   		  def almApplicationSystemChangeValue = almApplicationSystemChange?.newstring//if there is change almApplicationSystem value, Getting the new value updated to alm Application System field
                   		  if(almApplicationSystemChangeValue != ''){
                             logger.debug('Test'+almApplicationSystem)
                              log.warn(almApplicationSystem)
                             if(issueType == "Bug"){
                                    almApplicationSystemChangeValue = almApplicationSystemChangeValue.toString().replaceAll("\\s","")[0..2]
                                     logger.debug("Bug"+almApplicationSystem)
                             }
                            def almApplicationSystemChangeValueList = almApplicationSystemChangeValue.toString().replaceAll("\\s","").split(',') //Split the almApplicationSystem field value by ',' and replace extra space
                            updatePsIDSystem(almApplicationSystemChangeValueList,customFieldManager,psIdField,DefaultIssueChangeHolder,ModifiedValue,issue)
                          }else{
                               def almApplicationSystemChangeValueList = null
                               updatePsIDSystem(almApplicationSystemChangeValueList,customFieldManager,psIdField,DefaultIssueChangeHolder,ModifiedValue,issue)
                          }
                              
               }
            else if(eventTypeName == 'Issue Created' &&  issueType == 'Bug'){//Update the PSID and System field if new bug is created
                  almApplicationSystemValue = issue.getCustomFieldValue(almApplicationSystem)
                 // logger.debug(almApplicationSystem)
                if(almApplicationSystemValue != ''){//if alm Application System field is not null, extract the first three character from the field
                    almApplicationSystemValue = almApplicationSystemValue.toString().replaceAll("\\s","")[0..2]
                    def almApplicationSystemValueList = almApplicationSystemValue.toString().replaceAll("\\s","").split(',') //Split the Alm Application System field value by ',' and replace extra space
                    updatePsIDSystem(almApplicationSystemValueList,customFieldManager,psIdField,DefaultIssueChangeHolder,ModifiedValue,issue)//Calling updatePSIDSystem function
                }
            }
        }
                  
    }

}
//Update the System and PSID field using the value provided in alm Application System 
def updatePsIDSystem(def almApplicationSystemList, def customFieldManager, CustomField psIdField, def DefaultIssueChangeHolder, def ModifiedValue, Issue issue){
             
             def searchProvider = ComponentAccessor.getComponentOfType(SearchProvider.class);
			 def searchService = ComponentAccessor.getComponentOfType(SearchService.class);
             def userManager = ComponentAccessor.getUserManager()
   			 def issueManager = ComponentAccessor.getIssueManager()
    		 def issueService = ComponentAccessor.getIssueService()
			 def issueInputParameters = issueService.newIssueInputParameters()
        
             def appUser = userManager.getUserByName("Automation");
             List<String> systemsKey = new ArrayList<String>()
             List<Option> options = new ArrayList<Option>()
    		 def allOptions = ComponentAccessor.getOptionsManager()
					.getOptions(psIdField.getConfigurationSchemes().listIterator().next().getOneAndOnlyConfig());//Reading all the options of PSID field
             for (item in almApplicationSystemList){//Iterating over almApplicationSystemList
                        def optionToSelect = (allOptions.find{//finding the option for each alm Application System field value
                        it.value == item
                        })
                        //logger.debug("Test"+optionToSelect)
                        options.add(optionToSelect)//Adding the Options to Options List 

                        //Getting the SystemId for each of the PsId value by running the Jql
                        def jqlargument = "Project = System and \"Primary System Id\"~'"+item+"'"
                        SearchService.ParseResult parseResult = searchService.parseQuery(appUser,jqlargument);
                        Query query = parseResult.getQuery();
                        SearchResults results = searchService.search(appUser, query, PagerFilter.getUnlimitedFilter());
                        def fetchedIssues= results.getResults()
                        //logger.debug(fetchedIssues)

                        for ( issues in fetchedIssues ){//for each fetched issue, adding to the systemsKey array.
                             systemsKey.add(issues.getKey())
                             //logger.debug(systemsKey)
                        }
                }

                 
                //Creating string of System Key by reading the value from systemsKey array. 
                def count = 0
                def issueKeys = ''
                for (item in systemsKey){
                    if(count==0){
                        issueKeys = item
                        count=count+1
                    }
                    else
                     issueKeys =issueKeys+','+item//creating issueKeys string.
                    }

                //Updating the psIdField and Systems(s)
                //psIdField.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(psIdField), options), new DefaultIssueChangeHolder())
                //systesmcField.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(systesmcField), issueKeys), new DefaultIssueChangeHolder())
                def issueInputParametersValue =  issueInputParameters.addCustomFieldValue("customfield_15402",issueKeys)
                def validUpdateResult = issueService.validateUpdate(appUser,issue.id, issueInputParametersValue)
                if(validUpdateResult.isValid()){
                   issueService.update( appUser, validUpdateResult)					
                }
    
}


def clearEpic(Issue issue, CustomField epicLinkFiled ){//This function helps to clear the epic link 
     
     List<String> epicKey = new ArrayList<String>()
     def linkMgr = ComponentAccessor.getIssueLinkManager()
     def issueManager = ComponentAccessor.getIssueManager()
     def customFieldManager= ComponentAccessor.getCustomFieldManager()
     //def logger = Logger.getLogger("ConfluenceScript")
     if(issue.getCustomFieldValue(epicLinkFiled)!= null){//checking if epick link field is already null on test case issue
       epicLinkFiled.updateValue(null, issue, new ModifiedValue(issue.getCustomFieldValue(epicLinkFiled),null), new DefaultIssueChangeHolder())
     }
     def testIssuesLink = linkMgr.getInwardLinks(issue.id).findAll{ it.issueLinkType.id == 10003}//Getting the test issue linked to test case
     testIssuesLink.each{
          epicKey = []                            
         //Updating epic link on test issue
         if(it.getSourceObject().getCustomFieldValue(epicLinkFiled)!= null){//Checking if epick link field is already null on test issue
            epicLinkFiled.updateValue(null, it.getSourceObject(), new ModifiedValue(it.getSourceObject().getCustomFieldValue(epicLinkFiled), null), new DefaultIssueChangeHolder())       
         }                  
         def bugIssuesLink = linkMgr.getInwardLinks(it.sourceId).findAll{ it.issueLinkType.id == 10000}//Fetching bug issue
         
         bugIssuesLink.each{  
                        def bugIssue = it.getSourceObject()
                        def testIssueLinks = linkMgr.getOutwardLinks(bugIssue.id).findAll{ it.issueLinkType.id == 10000}//Getting all the test issue linked to bug
                        def testIssueEpicLinkFiledValue 
                        //logger.debug("Summary Bug issue"+bugIssue)
                        testIssueLinks.each{//Iterating over each test issue to get the primary epic.
                            def testIssue = it.getDestinationObject()
                            //logger.debug("Summary test issue"+testIssue)
                            testIssueEpicLinkFiledValue = testIssue.getCustomFieldValue(epicLinkFiled)//Getting the epic link assigned on Test issue
                            if(testIssueEpicLinkFiledValue!= null){//If epic link field is not empty
                                def primaryEpics = linkMgr.getOutwardLinks(issueManager.getIssueObject(testIssueEpicLinkFiledValue.toString()).id).findAll{ it.issueLinkType.id == 10307}//Get the primary epic linked to testing epic
                                if(primaryEpics.size()!= 0){//if primary epic is linked to testing epic
                                    def primaryEpic = primaryEpics[0].getDestinationObject()
                                    def primaryEpicKey = primaryEpic.getKey()
                                    epicKey.add(primaryEpicKey)//Add the primary key to the epic key.
                                }
                            }
                         }
                         //logger.debug("Summary epicKey"+epicKey)
                         if(epicKey.size() ==0 && bugIssue.getCustomFieldValue(epicLinkFiled)!= null){//If there is no test issue linked to bug and epic link field on bug is not already set to null                     
                             epicLinkFiled.updateValue(null, bugIssue, new ModifiedValue(bugIssue.getCustomFieldValue(epicLinkFiled),null), new DefaultIssueChangeHolder()) //Update the epic linkfield value on Bug issue
                         } 
                         else{
                             def winnerEpic = getEpic(epicKey,issueManager, customFieldManager,linkMgr) //If there are multiple test issue linked to different testing epic link
                             if(bugIssue.getCustomFieldValue(epicLinkFiled) != winnerEpic){//If the epic is currently linked to bug, is different from winner epic, then update the bug epic link
                                epicLinkFiled.updateValue(null, bugIssue, new ModifiedValue(bugIssue.getCustomFieldValue(epicLinkFiled), winnerEpic), new DefaultIssueChangeHolder())
                             }
                       }
                                               
        }
     }
}

/**GetEpic fundtion get the epic based on the following login
1. look at epic type via label value
   labels=FoundationalEpic  wins over labels=UseCaseEpic
2. look at Target End date; choose earliest end date
3. look at Rank; choose highest ranking Epic

**/
def getEpic(List epicKey, IssueManager issueManager,CustomFieldManager customFieldManager,IssueLinkManager linkMgr ){
    def rankingMap = [:]
    
    def List<Date> targetDatesList = new ArrayList<Date>()
    def List<String> rankList = new ArrayList<String>()
    def List<Integer> labelRank = new ArrayList<Integer>()
     
    
    
    def logger = Logger.getLogger("ConfluenceScript")
    //Iterating over the epicKey list to get the label, Target end Date and rank
    for (epics in epicKey){

        def List<String> stringLabels = new ArrayList<String>() 
        def epic = issueManager.getIssueByCurrentKey(epics.toString())
       
        
        def targetDate = customFieldManager.getCustomFieldObject("customfield_10203");//Getting Target Date
        def targetDateValue = epic.getCustomFieldValue(targetDate)
        if(targetDateValue != null){
            SimpleDateFormat df = new SimpleDateFormat("EEE MMM dd kk:mm:ss z yyyy",Locale.ENGLISH);
            Date date = df.parse(targetDateValue.toString());
            targetDatesList.add(date)//Adding TargetDate to the list
        }
        else{
            SimpleDateFormat df = new SimpleDateFormat("EEE MMM dd kk:mm:ss z yyyy",Locale.ENGLISH);
            Date date = df.parse("Mon jan 1 00:00:00 UTC 2050")

        }
        def rank = customFieldManager.getCustomFieldObject("customfield_10105");//Getting Rank
        def ranking = epic.getCustomFieldValue(rank).toString()[2..7]//Trunacating the rank first two character() usually the values look like  1|abc789:,   1|def111:)
        rankList.add(ranking)//Adding Rank to the list
        
        //Getting the Labels and assingin the rank to the epic based on the label found in epic.
        def labels = epic.getLabels()
        def customRank = 0
        for (item in labels){
            stringLabels.add(item.toString())
        }
        if('FoundationalEpic' in stringLabels){//If foundationalEpic found, assign rank 1
            customRank = 1
        }
        else if('UseCaseEpic' in stringLabels){//If UseCaseEpic found, assign rank 2
            customRank = 2
        }
        else{
            customRank = 3//Else assign rank 3
        }
        labelRank.add(customRank)
    }
    
    //logger.debug(labelRank)
    //logger.debug(targetDatesList)
    //logger.debug(rankList)
    def winnerEpic = ""
    def count = 0
    def highPriorityLabel = 0
    Date oldTargetDate  
    def rank = ""
    
    
    for(int i=0; i<epicKey.size(); i++){
       
        if(count == 0){//Initialize the winner epic for the first iteration
         
            winnerEpic = epicKey[i]
            highPriorityLabel = labelRank[i]
            oldTargetDate = targetDatesList[i]
            rank = rankList[i]
            
        }
        else{
     		
            if(labelRank[i] < highPriorityLabel){//Compare the label rank and if the label rank for the current item is higher then the previous item, assign the current epic to Winner Epic
                //logger.debug('labelRank comparison labelRank[i] < highPriorityLabel')
                
                highPriorityLabel = labelRank[i]   
                winnerEpic = epicKey[i]
                oldTargetDate = targetDatesList[i]
           	    rank = rankList[i]
           
                
            }
            else{
                if(labelRank[i] == highPriorityLabel){//Else if rank is equesl for current and previous item, compare the target end date
                   
                    if(targetDatesList[i] < oldTargetDate ){//select the epic with the recent target end date
                  
                        oldTargetDate =targetDatesList[i]
                        winnerEpic = epicKey[i]
                        highPriorityLabel = labelRank[i] 
                        rank = rankList[i]
                        
                        
                    }
                    else{
                        if (targetDatesList[i] == oldTargetDate){//Else if target end date is same for previous and current item, compare the rank. 
                            if(rank < rankList[i]){//Comparing the rank
                                
                                rank = rankList[i]
                                oldTargetDate =targetDatesList[i]
                                winnerEpic = epicKey[i]
                                highPriorityLabel = labelRank[i] 
                                
                            }	
                        }
                    }
                }
            }
           
            
        }
        count = count+1
        //logger.debug("winnerEpic" +winnerEpic)
    }

    //Once the primary epic if select, Get the testing epic
    def primaryEpic = issueManager.getIssueByCurrentKey(winnerEpic.toString())
    //logger.debug("primaryEpic" +primaryEpic)
    if(primaryEpic!= null){
        def testEpics = linkMgr.getInwardLinks(primaryEpic.id).findAll{it -> it.issueLinkType.id == 10307}
        //logger.debug("testEpics" +testEpics)
        //logger.debug("testEpics" +testEpics.size())
        if(testEpics.size() != 0){
        def testEpic
        testEpics.each{ testEpic = it.getSourceObject().getKey()}
         return  issueManager.getIssueByCurrentKey(testEpic.toString())
        }
	}
  
  
 }

/////////////////////////////////////////////////////////////////////////////////////

LISTNER PSID AND ACRONYM

import com.atlassian.jira.issue.Issue
import com.atlassian.jira.issue.ModifiedValue
import com.atlassian.jira.issue.util.DefaultIssueChangeHolder
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.event.type.EventDispatchOption
import com.atlassian.jira.issue.MutableIssue
import com.atlassian.jira.issue.index.IssueIndexingService
import com.atlassian.jira.util.ImportUtils

def eventTypeManager = ComponentAccessor.getEventTypeManager()
def eventTypeName = eventTypeManager.getEventType(event.eventTypeId).getName()
	//log.error("eventTypeName "+ eventTypeName) 

def artifactChange = event?.getChangeLog()?.getRelated("ChildChangeItem")?.find {it.field == "Artifact"}
def systemsChange = event?.getChangeLog()?.getRelated("ChildChangeItem")?.find {it.field == "System(s)"}
def parentChange = event?.getChangeLog()?.getRelated("ChildChangeItem")?.find {it.field == "Parent Issue"}
def storyToSTChange = event?.getChangeLog()?.getRelated("ChildChangeItem").find {it.field == "Parent"}
def maintenanceChange = event?.getChangeLog()?.getRelated("ChildChangeItem")?.find {it.field == "Maintenance"}

if ((artifactChange) || (systemsChange) || (eventTypeName == "Issue Created") || (parentChange) || (storyToSTChange) || (maintenanceChange)) {

    def issue = event.issue as Issue
	MutableIssue issueToUpdate = (MutableIssue) issue
		//log.error('key: ' + issue.key) 
    def customFieldManager = ComponentAccessor.getCustomFieldManager()
	def optionsManager = ComponentAccessor.getOptionsManager()
	def issueManager = ComponentAccessor.getIssueManager()
	def issueIndexingService = ComponentAccessor.getComponent(IssueIndexingService)
	boolean wasIndexing = ImportUtils.isIndexIssues();

//UPDATE ARTIFACT ACRONYM & PHASE IF ARTIFACT IS UPDATED     
    if ((artifactChange) || (eventTypeName == "Issue Created") || (maintenanceChange?.newstring == "art") ) {

		def ArtifactAcr
		def PhaseID
		def cfArtifactcwx = customFieldManager.getCustomFieldObjectsByName("Artifact")[0] // getting Artifact field from issue type Artifact
		def cfArtifactAcronym = customFieldManager.getCustomFieldObjectsByName("Acronym for Artifact Record")[0] // getting Artifact Acronym field from issue type Artifact
		def cfArtifactAcr = customFieldManager.getCustomFieldObjectsByName("Artifact Acronym")[0] // getting ArtAcronym field from issue type Story
		def cfPhaseID = customFieldManager.getCustomFieldObjectsByName("Phase")[0]  // getting PhaseID field from issue type Story

		def cfConfig1 = cfArtifactAcr.getRelevantConfig(issue)
		def cfConfig2 = cfPhaseID.getRelevantConfig(issue)

		List artifactList = new ArrayList<>()
		List phaseList = new ArrayList<>()
		def changeHolder = new DefaultIssueChangeHolder()

		//update ArtAcronym, Phase
		def artifactVal = issue.getCustomFieldValue(cfArtifactcwx)
			//log.error('artifactVal: ' + artifactVal)
		if (artifactVal) {
    		def artifact = artifactVal.toString().split(',')
				//log.warn('Artifact: ' + artifact)

			artifact.each { art ->
				if (art) {
   					//log.warn('art: '+art)
   					ArtifactAcr = issueManager.getIssueObject(art.toString()).getCustomFieldValue(cfArtifactAcronym) // copying value of Artifact Acronym to ArtAcronym
       			 	PhaseID = issueManager.getIssueObject(art.toString()).getCustomFieldValue(cfPhaseID)// copying value of Phase to phaseID
    					//log.error('ArtifactAcr:'+ ArtifactAcr.toString())
            			//log.warn('PhaseID:'+ PhaseID.toString())
                
    				def artAcronymValue = optionsManager.getOptions(cfConfig1)?.find {
                		it.toString() == ArtifactAcr}
            
            		def phaseValue = optionsManager.getOptions(cfConfig2)?.find{
                		it.toString() == PhaseID.toString()}
            				//log.warn('ArtifactAcrValue: '+ artAcronymValue)
            				//log.warn('phaseValue: '+ phaseValue)
    				if (artAcronymValue) {
    					artifactList.add(artAcronymValue) 
    				}    
            
            		if (phaseValue) {
    					phaseList.add(phaseValue)
    				} 
            			//log.error('artifactList: '+ artifactList.toString())
            			//log.error('phaseList: '+ phaseList.toString())
    				}
			}
		}    
			//log.warn('artifactListfull: '+artifactList.toString())
			//log.warn('phaseListfull: '+phaseList.toString())

	def tgtFieldArt = customFieldManager.getCustomFieldObjects(event.issue).find {it.name == "Artifact Acronym"}
	def tgtFieldPha = customFieldManager.getCustomFieldObjects(event.issue).find {it.name == "Phase"}
		//log.warn('tgtFieldPha: '+ tgtFieldPha)

	changeHolder = new DefaultIssueChangeHolder()
	tgtFieldArt.updateValue(null, issueToUpdate, new ModifiedValue(issue.getCustomFieldValue(tgtFieldArt), artifactList),changeHolder) // Update Artifact
	tgtFieldPha.updateValue(null, issueToUpdate, new ModifiedValue(issue.getCustomFieldValue(tgtFieldPha), phaseList[0]),changeHolder) // Update Phase
	/*
    //Reindex
	ImportUtils.setIndexIssues(true);
		//log.warn("Reindex issue ${issue.key} ${issue.id}")
	issueIndexingService.reIndex(issueManager.getIssueObject(issue.id))
	ImportUtils.setIndexIssues(wasIndexing) 
    */    
	} //End artifactChange
    
    
//UPDATE PSID IF SYSTEM(S) IS UPDATED 
	// updated to exclude Sub-tasks, as this functionality was clearing PSID on newly created Sub-tasks after post-function on Create
    // Transition of Workflow synced PSID w/ parent of Sub-task  -- RT 08/13/20
    if (((systemsChange) || (storyToSTChange?.oldstring) || (eventTypeName == "Issue Created") || (maintenanceChange?.newstring == "sys")) && issue.getIssueType().getName() != "Sub-task") {
    	
	def PSID
	def value
	def cfSystemscwx = customFieldManager.getCustomFieldObjectsByName("System(s)")[0]
	def cfPrimSysID = customFieldManager.getCustomFieldObjectsByName("Primary System ID")[0]
	def cfPSID = customFieldManager.getCustomFieldObjectsByName("PSID")[0]
	def cfConfig = cfPSID.getRelevantConfig(issue)
	def customFieldOptions = ComponentAccessor.optionsManager.getOptions(cfConfig)
	def cfInterfaces = customFieldManager.getCustomFieldObjectsByName("Interface(s)")[0]
	def interfacesArray = issue.getCustomFieldValue(cfInterfaces).toString().split(',') //Split field values by , into an array
		//log.info('issue: ' +issueToUpdate.key.toString())
		//log.warn('interfacesArray: ' + interfacesArray[0].toString())
    
	List systemVal = new ArrayList<>()
	List PSIDval = new ArrayList<>()   
	def changeHolder = new DefaultIssueChangeHolder()
	def tgtSystems = customFieldManager.getCustomFieldObjects(event.issue).find {it.name == "System(s)"}
	def tgtPSID = customFieldManager.getCustomFieldObjects(event.issue).find {it.name == "PSID"}
    
    //Update PSID ONLY
	List systemsList = new ArrayList<>()  //Array to store selected Systems
	def systemsVal = issue.getCustomFieldValue(cfSystemscwx) ////Get value(s) from Systems field
		//log.warn('systemsVal: ' + systemsVal)

	if (systemsVal) { //Test if there is Ss
    	//log.warn('In: ' + systemsVal)
    	def systems = systemsVal.toString().split(',')
			//log.warn('System(s): ' + systems)
		systems.each { sys ->
        	if (sys) {
   				PSID = issueManager.getIssueObject(sys.toString()).getCustomFieldValue(cfPrimSysID)
    				//log.warn(PSID.toString())
    			value = ComponentAccessor.optionsManager.getOptions(cfConfig)?.find {
    				it.toString() == PSID}
    					//log.warn('PSID: '+ value)
    			if (value){
    				systemsList.add(value)
    			}
            } 
    	}
	}
		//log.warn('systemlistfull: '+systemsList.toString())

	changeHolder = new DefaultIssueChangeHolder()
	tgtPSID.updateValue(null, issueToUpdate, new ModifiedValue(issue.getCustomFieldValue(tgtPSID), systemsList),changeHolder)
	/*
        //Reindex
		ImportUtils.setIndexIssues(true);
			//log.warn("Reindex issue ${issue.key} ${issue.id}")
		issueIndexingService.reIndex(issueManager.getIssueObject(issue.id));
		ImportUtils.setIndexIssues(wasIndexing);

	*/
    Collection<Issue> subTasks = issueToUpdate.getSubTaskObjects()
	//Iterate through Sub-tasks
	subTasks.each { st ->
    	if (st instanceof MutableIssue) {
            	// changed references to issueToUpdate and issue to be st, so that value on Sub-task is updated -- RT 08/13/20
				tgtPSID.updateValue(null, st, new ModifiedValue(st.getCustomFieldValue(tgtPSID), systemsList),changeHolder)
        		//Reindex
        		ImportUtils.setIndexIssues(true);
					//log.warn("Reindex issue ${it.key} ${it.id}")
				issueIndexingService.reIndex(issueManager.getIssueObject(st.id));
				ImportUtils.setIndexIssues(wasIndexing);
    	}
	}   
    } //End systemsChange

// SYNC SUB-TASK PSID WITH STORY PSID IF PARENT ISSUE IS CHANGED    
    if (((parentChange) || (storyToSTChange?.newstring) || (maintenanceChange?.newstring == "subt")) && (eventTypeName != "Issue Created")) {
        def cfPSID = customFieldManager.getCustomFieldObjectsByName("PSID")[0]
        def parentPSIDVal = cfPSID.getValue(issue.getParentObject())
        
        //log.warn("parentPSIDVal = " + parentPSIDVal)
        
        cfPSID.updateValue(null, issue, new ModifiedValue(cfPSID.getValue(issue), parentPSIDVal), new DefaultIssueChangeHolder())
    }  //End Sub-task Parent Issue Change PSID Sync
    
    if (storyToSTChange?.newstring) { return }
    	//Reindex in all cases except Story changed to Sub-task (throws error)
		ImportUtils.setIndexIssues(true);
			//log.warn("Reindex issue ${issue.key} ${issue.id}")
		issueIndexingService.reIndex(issueManager.getIssueObject(issue.id));
		ImportUtils.setIndexIssues(wasIndexing);
    
  
} //End Listener



////////////////////////////////////////////////////////////////////////////////////////
rest endpoint


import com.onresolve.scriptrunner.runner.rest.common.CustomEndpointDelegate
import groovy.json.JsonBuilder
import groovy.transform.BaseScript
import com.atlassian.jira.component.ComponentAccessor;
import com.atlassian.jira.bc.issue.search.SearchService
import com.atlassian.jira.web.bean.PagerFilter
import com.atlassian.query.Query
import com.atlassian.jira.issue.search.SearchResults
import com.atlassian.jira.issue.search.SearchProvider
import com.atlassian.jira.issue.search.SearchQuery
import java.text.SimpleDateFormat;
import com.atlassian.jira.issue.link.IssueLinkManager
import com.atlassian.jira.issue.link.IssueLinkTypeManager
import javax.ws.rs.core.MediaType
import com.atlassian.jira.user.ApplicationUser
import org.apache.log4j.Logger
import org.apache.log4j.Level
import javax.ws.rs.core.MultivaluedMap
import javax.ws.rs.core.Response

@BaseScript CustomEndpointDelegate delegate

getInterfaceKey(httpMethod: "GET", groups: ["CBT users"]) { MultivaluedMap queryParams, String body ->

def dateRange = queryParams.getFirst("since").toString()//Reading the Date parameters from the end point url
def logger = Logger.getLogger("ConfluenceScript")
def appUser = ComponentAccessor.getJiraAuthenticationContext().getLoggedInUser()
def userManager = ComponentAccessor.getUserManager();
def linkMgr = ComponentAccessor.getIssueLinkManager()
def changeHistory = ComponentAccessor.getChangeHistoryManager()
def issueManager = ComponentAccessor.getIssueManager()
def optionsManager = ComponentAccessor.getOptionsManager()
def searchProvider = ComponentAccessor.getComponentOfType(SearchProvider.class);
def searchService = ComponentAccessor.getComponentOfType(SearchService.class);
//def eventTypeManager = ComponentAccessor.getEventTypeManager()
def customFieldManager = ComponentAccessor.getCustomFieldManager()
def issueLinkTypeManager = ComponentAccessor.getComponent(IssueLinkTypeManager)
    
List<String> issuestory = new ArrayList<String>()
List<String> interfacestory = new ArrayList<String>()
def df = new SimpleDateFormat("EEE MMM dd kk:mm:ss z yyyy", Locale.ENGLISH);
def  jqlargument = ''
//Initializing the cusotmfields
def accountablePM = customFieldManager.getCustomFieldObject("customfield_13300");
def targetEndDate =  customFieldManager.getCustomFieldObject("customfield_10203");
def artifact = customFieldManager.getCustomFieldObject("customfield_16202");
def phase = customFieldManager.getCustomFieldObject("customfield_10714");
    
/**Building a Dialog Html as stringbuilder **/
/**Building a Dialog Html as stringbuilder **/
StringBuilder result = new StringBuilder()
if(Integer.parseInt(dateRange) <=30){
      jqlargument = 'project = cbt AND issuetype = story AND status was in (Open)  DURING (startOfDay(-'+dateRange+'), startOfDay()) AND "Artifact Acronym" in (DEV, REN) AND status not in (Open, Cancelled) and "Interface(s)" is not EMPTY and issueFunction in linkedIssuesOf("project = interfaces and \'Related Development\' is not EMPTY  ")'
	   result.append(
        """<section role="dialog" id="sr-dialog" class="aui-layer aui-dialog2  aui-dialog2-xlarge" aria-hidden="true" data-aui-remove-on-hide="true">
            <header class="aui-dialog2-header">
                <h2 class="aui-dialog2-header-main">List of Interface(s)</h2>
                <a class="aui-dialog2-header-close">
                    <span class="aui-icon aui-icon-small aui-iconfont-close-dialog">Close</span>
                </a>  
            </header>
            <div class="aui-dialog2-content">
                <P><b>Below are the list of Impacted Interface(s) caused by addition of dev item where prior dev items were completed.</b> </p>
                <p></p>
                <table style=\"border: 1px solid #808080; border-spacing: 0px;\">
                <tr>
                    	<th style=\"border-right: 1px solid #808080; padding:10px\">Interface Key</th>
                        <th style=\"border-right: 1px solid #808080; padding:10px\">Interface Name</th>
                        <th style=\"border-right: 1px solid #808080; padding:10px\">Story</th>
                        <th style=\"border-right: 1px solid #808080; padding:10px\">Story Name</th>
                        <th style=\"border-right: 1px solid #808080; padding:10px\">Story ‘Created BY’</th>
                        <th style=\"border-right: 1px solid #808080; padding:10px\">Story ‘Created Date’</th>
                        <th style=\"border-right: 1px solid #808080; padding:10px\">Story ‘Accountable PM’</th>
                        <th style=\"border-right: 1px solid #808080; padding:10px\">Story ‘Target End Date’</th>
                        <th style=\"border-right: 1px solid #808080; padding:10px\">Story ‘Status’</th>
                        <th>Story ‘Artifact’</th>
                 </tr>
            """)
}
else
{
    jqlargument = 'key = cbt-321'
    result.append('''<section role="dialog" id="sr-dialog" class="aui-layer aui-dialog2  aui-dialog2-xlarge" aria-hidden="true" data-aui-remove-on-hide="true">
            <header class="aui-dialog2-header">
                <h2 class="aui-dialog2-header-main">List of Interface(s)</h2>
                
            </header>
            <div class="aui-dialog2-content">
            <P><b>Try again for Filter less then 30 Days.</b> </p>
            </div>''')
}  
//Executing the JQL

//def jqlargument = "key = INTERFACE-8544"
SearchService.ParseResult parseResult = searchService.parseQuery(appUser,jqlargument);
Query query = parseResult.getQuery();
SearchResults results = searchService.search(appUser, query, PagerFilter.getUnlimitedFilter());
def fetchedIssues= results.getResults()
 
//Getting the current date
def today = new Date()
def dateRangeHistory = today-Integer.parseInt(dateRange)//Getting the date range by subtracting the current date - data provided in the url as parameter
Date sinceDate =  Date.parse("yyyy-MM-dd HH:mm:ss", dateRangeHistory.format("yyyy-MM-dd HH:mm:ss"))//Converting the daterangeHistory to date format

//Iterating over the fetched issue and reading the changehistory for the ready story Item
fetchedIssues.each{
    def issue = issueManager.getIssueByCurrentKey(it.key)
    //logger.debug("Insideissue"+issue)
    def storyStatus = issue.getStatus()
    //logger.debug("Insideissue"+storyStatus)
    if(storyStatus.name == 'Ready'){
        def  history = changeHistory.getChangeHistoriesSince(issue, sinceDate)//Fetching the change history item from the sincedate
        // logger.debug("Test"+sinceDate+history)
        history.each{
               it.getChangeItemBeans().each{
                   if(it.field == 'status' && it.toString == 'Ready' && it.fromString == 'Open'){//If there is change in status from Open to Ready with in the since date add the item to the issueStory List
                     issuestory.add(issue.getKey())
                       //logger.debug("Test"+issuestory)
                   }
              }
        }
        history.each{
               it.getChangeItemBeans().each{//If the item was moved from ready status to any other status then remove it from the list to back to Open
                   if(it.field == 'status' && it.fromString == 'Ready' && it.toString == 'Open'){
                     issuestory.remove(issue.getKey())
                    
                   }
              }
        }
    }             
}

issuestory.each{//Iterating over each story and checking if the interface linked to dev item and status of dev items.
     def issue = issueManager.getIssueByCurrentKey(it)
     def interfaceItem = linkMgr.getOutwardLinks(issue.id).findAll{ it.issueLinkType.id == 12300}
     interfaceItem.each{
        
        def interfaceIssue = it.getDestinationObject()
        def workItems = linkMgr.getInwardLinks(interfaceIssue.id).findAll{ it.issueLinkType.id == 12300}//Getting the workitem
        //def count = 0  
        def devCount = 0
        def designCount = 0
        def doneDesignCount = 0
        def notDevCount = 0
        def donedevcount = 0
       
        workItems.each{//Foreach workitem , checking the status and it has dev artifact and getting count of how many dev done items and dev ready items and non dev items
            def devArtifactFlag = false
        	def workitem = it.getSourceObject()
              // logger.debug(workitem)
              // logger.debug(issue)
            if(workitem.getKey().toString() != issue.getKey().toString()){
               // logger.debug("Inside Workitem")
                def status = workitem.getStatus()
                def summary = workitem.getSummary()
                def devLabel = workitem.getLabels()
                def devArtifact = workitem.getCustomFieldValue(artifact)
                def phaseValue = workitem.getCustomFieldValue(phase)
                devArtifact.each{
                    if(it.toString() == 'DEV' ||it.toString() == 'REN'  ||it.toString() == 'WBK-Dev'  ){
                          devArtifactFlag = true
                         //logger.debug(it)
                     }
                }   
                if(status.name == 'Done' &&  devArtifactFlag == true){
                   devCount = devCount +1
                    donedevcount = donedevcount + 1
                    //logger.debug("status.name == 'Done' &&  devlabel == true")
                    //logger.debug(devCount)  
                }
              
                /*else if(status.name == 'Done' &&  devArtifactFlag == false){
                    //logger.debug("status.name == 'Done' &&  devlabel == false")
                    notDevCount = notDevCount +1
               }
               else if(status.name == 'Cancelled' &&  devArtifactFlag == false){
                    //logger.debug("status.name == 'Cancelled' &&  devlabel == false")
                    notDevCount = notDevCount +1
               }
                else if(status.name == 'In Progress' &&  devArtifactFlag == false) {
                    // logger.debug("status.name == 'In Progress' &&  devlabel == false")
                      notDevCount = notDevCount +1
                }*/
                else if(status.name == 'Open' &  devArtifactFlag == true) {
                     // logger.debug("status.name == 'Open'")
                      notDevCount = notDevCount +1
                }
                 else if(devArtifactFlag == false ) {
                     // logger.debug("status.name == 'Open'")
                      notDevCount = notDevCount +1
                }
                /*else if(status.name == 'Ready' &&  devArtifactFlag == false) {
                     // logger.debug("status.name == 'Open'")
                      notDevCount = notDevCount +1
                }*/
                else if(status.name == 'Ready' &&  devArtifactFlag == true) {
                    //logger.debug("status.name == 'Ready' &&  devlabel == true")
                    def  history = changeHistory.getChangeHistoriesSince(workitem, sinceDate)
                    history.each{
                           it.getChangeItemBeans().each{
                               if(it.field == 'status' && it.toString == 'Ready' && it.fromString == 'Open'){
                                 devCount  = devCount +1
                                   //logger.debug("status.name == 'Ready' &&  devlabel == true")
                                     //logger.debug(  it)

                               }
                          }
                    }
                }
               
            }
            else{
              
                devCount = devCount +1
            }
        }
         /*logger.debug("devCount:"+devCount)
         logger.debug("donedevcount:"+donedevcount)
         logger.debug("doneDesignCount:"+doneDesignCount)
         logger.debug("designCount:"+designCount)
         logger.debug("notDevCount:"+notDevCount)
         logger.debug("count:"+count)*/ 
        // logger.debug(workItems.size())
   
          if(devCount + notDevCount == workItems.size() &&( devCount >1 && donedevcount !=0)){//if dev count is more then 1 and donedevCount is not zero
             interfacestory.add(interfaceIssue.key)//Add the interface to InterfaceStory
             
             def targetEndDateStory = issue.getCustomFieldValue(targetEndDate)//Reading the Target end Date for the issue
 			 Date dt =  df.parse(targetEndDateStory.toString());//Converting the TargetEnd Date to Date
			 def targetEndDateFormat = dt.format("yyyy-MM-dd HH:mm:ss")//Converting TargetEndDateFormat to the given format
             ApplicationUser accountPM = (ApplicationUser)issue.getCustomFieldValue(accountablePM)//Getting the Accountable PM
			 def devArtifact = issue.getCustomFieldValue(artifact)//Getting the devArtifact
             def devArtifactAcronym = ''
             devArtifact.each{
                  devArtifactAcronym = devArtifactAcronym + it.toString()
             }
			  //Creating the Table 
              result.append("<tr><td style=\"border-right: 1px solid #808080; border-top: 1px solid #808080; padding:10px\"><a target=\"_blank\" href=https://prod-1-jira.mufgamericas.com/browse/${interfaceIssue.key}\">").append("${interfaceIssue.key}")
              result.append("</td><td style=\"border-right: 1px solid #808080; border-top: 1px solid #808080; padding:10px\">").append("${interfaceIssue.summary}")
              result.append("</td><td style=\"border-right: 1px solid #808080; border-top: 1px solid #808080; padding:10px\"><a target=\"_blank\" href=https://prod-1-jira.mufgamericas.com/browse/${issue.getKey()}\">").append("${issue.getKey()}")
              result.append("</td><td style=\"border-right: 1px solid #808080; border-top: 1px solid #808080; padding:10px\">").append("${issue.getSummary()}")
              result.append("</td><td style=\"border-right: 1px solid #808080; border-top: 1px solid #808080; padding:10px\">").append("${issue.creator.displayName}")
              result.append("</td><td style=\"border-right: 1px solid #808080; border-top: 1px solid #808080; padding:10px\">").append("${issue.created.format("yyyy-MM-dd HH:mm:ss")}")
              result.append("</td><td style=\"border-right: 1px solid #808080; border-top: 1px solid #808080; padding:10px\">").append("${accountPM.getDisplayName()}")
              result.append("</td><td style=\"border-right: 1px solid #808080; border-top: 1px solid #808080; padding:10px\">").append("${targetEndDateFormat}")
              result.append("</td><td style=\"border-right: 1px solid #808080; border-top: 1px solid #808080; padding:10px\">").append("${issue.status.name}")
              result.append("</td><td style=\"border-right: 1px solid #808080; border-top: 1px solid #808080; padding:10px\">").append("${devArtifactAcronym}")
              result.append("</tr></div>")
    
         }
    }
    
}
      if(Integer.parseInt(dateRange) <=30){
        result.append("<p>Total number of Interface(s) impacted are "+interfacestory.size()+" in last  "+dateRange+"  days.</p><p></p>")//Appending the Item to Dialog
        result.append(""" </table>
                <div>
                    <p></p>
                    <br>
                        <a class="btn btn-success" href="https://prod-1-confluence.mufgamericas.com/pages/viewpage.action?pageId=158031562" target="_blank">Click here for more Filters.</a>
                </div>
                </div>
                <!-- Dialog footer -->
                <footer class="aui-dialog2-footer">
                
                    <div class="aui-dialog2-footer-actions">
                    <button id="dialog-close-button" class="aui-button aui-button-link">Close</button>
                    </div>
                
            </footer>
        </section>
        """)
    }
    
   // return Response.ok().type(MediaType.TEXT_HTML).entity(result.toString()).build()
   return Response.ok().type(MediaType.TEXT_HTML).entity(result.toString()).build()
}
/////////////////////////////////////////////////////////////////////////////////////







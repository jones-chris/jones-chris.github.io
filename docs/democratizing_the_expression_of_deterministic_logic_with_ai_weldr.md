# Democratizing the Expression of Deterministic Logic with AI Weldr

![claude integration](./images/ai_weldr_claude_result.png)

## Summary / Intro

Non-technical business users are often intimately familiar with their business processes.  However, they lack the technical programming skills to express that business process in a way that computers can efficiently execute.

Due to this limitation, the path for a business user to express their deterministic logic and automate it is lengthy, such as this:

They usually will put together a presentation and lobby for budget approval to their department leads or perhaps at their annual company-wide days-long budget planning 
meetings.

Assuming they get budget approval, IT employees become involved:  A project manager or business analyst liaisons with the business users to understand their requirements.  A team of software engineers is assigned to design 
and build the necessary system.  There are multiple meetings involving these groups to gather requirements and later to perform user acceptance testing as they iterate on the solution.

Understandably, this expensive process – in time, people, and money – favors funding projects that have a high return, which are usually complex projects.

This leaves business users with simple or medium-complexity projects left to a self-service methodology.  For example, perhaps the organization obtains funding for low-code SaaS licenses, such as Alteryx.  Some – but not 
all - users will be motivated to learn how to use the SaaS product sufficiently to automate their use cases, but they will be locked into the SaaS licenses indefinitely.  However, most users will not obtain the funding or 
have the desire to learn technical SaaS products to address their use cases.

Even with the advent of AI in recent years, these users may be able to use AI to generate a simple app, but they still need to know a fair amount about programming languages to maintain and refactor the app.  Moreover, the
app is confined to their laptops and is not deployed to servers where it can have more resources or can be scheduled to run.  

Certainly the funding and IT resources are necessary for high performance, complex, mission-critical systems, but what about the simple and medium-complexity use cases?  Would the sum of all those use cases result in 
meaningful productivity gains and digital transformation if only there was a way to overcome the barriers of funding and technical expertise that prevent non-technical business users from expressing the business logic they 
understand better than anyone else in the organization?

I propose that there is a way to allow users to do this using tools that they may already use every day – or can learn more easily than a programming language – at extremely low cost, even free for low-traffic use cases. AI 
can still help them build it, but the tooling avoids programming languages, so they can better understand, maintain, and refactor the application.  The answer involves 1) spreadsheets (specifically the open XLSX file format), 2) serverless 
cloud provider services, and 3) an open source library called Apache POI.

## Building a Reusable Artifact 

Spreadsheets are core to this solution.  Virtually every business user has some knowledge of how to write formulas to express business logic whether it be summing two numbers, concatenating text, or something more complex
like joining two datasets based on IDs.  Virtually every user has a spreadsheet software on their laptop, whether it requires a paid license like Microsoft Excel, free options like Google Sheets, or open source options like LibreOffice.  Many of these products share the same formulas and feature sets.

In addition, since 2007 spreadsheets can be saved in the XLSX file format, which is an open format that compresses XML files.  You can verify this for yourself – unzip a XLSX file and you’ll find a bunch of XML files.  Moreover, you may
have noticed that a spreadsheet you created in Microsoft Excel and saved as a XLSX file can be opened in another spreadsheet software like LibreOffice with minimal or no changes.

XLSX is the lingua franca of spreadsheet tools today and it’s entirely open.  With it, users have multiple paid, free, and open source products to express their business processes in formula syntax that can be executed in
these products.  Importantly, these products also allow users to debug their business logic.  If input data causes a formula calculation error, a user can use any of these products to open the XLSX file and troubleshoot 
the issue without IT needing to be involved.

However, products like Microsoft Excel, Google Sheets, and LibreOffice are difficult if not impossible to run on servers.  But, on the other hand, the mature and open source Apache POI project includes a calculation engine
that can read and write XLSX files and execute its formulas.  Apache POI is a Java library, which means we can use it as the bridge between the business logic expressed in an XLSX file and a critical, mature traditional 
application runtime: the Java Virtual Machine or JVM, which runs some of the most important applications we use today.

Now that we’ve covered the concepts and tools we’ll use to build a XLSX-powered application, let’s understand how they come together to allow users to run their XLSX business logic.

The key is keeping our business users in spreadsheets where they can express their business logic in formulas.  We don’t want them to have to know anything about Apache POI or Java.  

To do this, a reusable executable Java application that “wraps” around an XLSX file.  It will load the XLSX file into memory from JAR’s `/resources` directory (a standard Java application folder), stream input datasets 
into it, calculate the formulas, and write the resulting XLSX file to storage or return certain values back to the client or user.

To make it easy for non-technical users to load their XLSX file into this reusable executable Java application there is a small web page where they can upload their XLSX file.  The web page returns the JAR so they can 
download it and run it.

## Running Your JAR: Options for Every Skill Set

Users now have a JAR that can run their XLSX file.  I think they have 4 options to run it.  Each option is detailed below and starts with the simplest option and progresses to the most complex option.  Each has advantages
and disadvantages.

### Option 1: Local MCP Server Integration (simplest, AI-assisted) 

The reusable executable JAR includes an MCP server.  MCP servers expose their logic to AI models so that the AI models can run the JAR.

Assuming the user uses Claude Desktop, they would make one change to Claude’s MCP servers configuration:

```json
{
  "mcpServers": {
    "myXlsxJar": {
      "command": "java",
        "args": [
            "-jar",
            "/path/to/my/xlsx.jar"
        ]
    }
  }
}
```

In the above example, the user would replace “myXlsxJar” with a descriptive name of what the XLSX does (ex: customerInformation if the XLSX joins customer datasets) and "/path/to/my/xlsx.jar" with the absolute file path 
to the JAR.

After doing so, Claude will now start the MCP server and can interact with it as needed based on the user’s natural language prompts.

If the XLSX encounters an error when its formulas are calculated, the user can debug it by simply opening the XLSX file, making the necessary change, and uploading the XLSX file to create a new JAR.  There is no need to 
get IT involved – the troubleshooting process is entirely in the user’s control.

This reduces the technical knowledge the user needs to run their business logic.

### Option 2: Team MCP Server Shared Drive (collaborative, still local)

This option builds on Option 1 by creating a shared drive so that users in the same organization or team can share their JARs with each other.  Team members can then download the necessary JARs and add them to their MCP 
server configurations.

This benefits the team by not reinventing the same JAR and using AI to “chain” JARs together.  For example, a user may download 2 JARs:  the first calculates revenue and the second calculates taxes.  The user could then
write one prompt for the AI model to calculate revenue and then calculate taxes.

### Option 3: Cloud Deployment (scales compute resources)

If a user has a large dataset that requires more compute resources than what is available on their local machine, they can deploy the JAR to a cloud provider’s serverless services, such as AWS Lambda, which has a maximum
memory of 10GB.

I’m still building this option but currently think users could upload their input dataset files to an S3 bucket, which triggers the Lambda Function and writes the results to another S3 bucket.  The user can then retrieve
the entire file or just request certain named ranges from it.

It’s important to understand that while spreadsheet software like Microsoft Excel is limited to 1,048,576 rows per sheet, Apache POI’s streaming capabilities (SXSSF) allow you to handle datasets  larger than what a 
standard desktop can process. By running your logic in the cloud, you can accommodate massive inputs that would crash a local machine. However, there is a trade-off: If your resulting XLSX file exceeds the standard 
1-million-row limit per sheet, you may lose the ability to open and debug that file in desktop software, as it will violate the official XLSX specification.

### Option 4: Cloud with REST API Scripts (programmatic, for technical business users)

If a user wanted to automate the process entirely, as an example they could schedule a script to export the necessary datasets and send them to the S3 bucket in Option 3 above, which would trigger the JAR to run.  

Such a script would need to be customized based on the datasets the user needs.  This may require IT to be involved or the user to have sufficient technical skills to write the script themselves or with the help of AI.

### The AI Weldr Specification:  How to Write XLSX Files for AI Weldr

In order for the JAR to correctly wrap the XLSX file, the XLSX file must follow some standards:

  1. There must be a sheet called `metadata`.
  2. In the `metadata` sheet, there must be the following named range cells:
        1. errors:  This is a count of all errors in the spreadsheet that signify something went wrong in the calculation. If 0, the JAR interprets this as the calculation succeeded. If not 0, the JAR interprets thisthis 
        as the calculation failed. For IT audiences, you can think of this as similar to shell command exit codes. For example, the errors cell formula may be `=SUMPRODUCT(--ISERROR($output_0.A1:D5000))`
        2. author: The name of the author of the spreadsheet.
  3. The workbook contains zero or more named ranges that begin with the prefix `input_`. These named ranges are where the input datasets will be placed. They will be exposed to clients such as AI models. Note that
     care should be taken when setting the size of the named range because if the size of the input dataset is greater than the named range size, then the JAR will throw an error. Also, it is optional but recommended that you assign data types to each column of the named range.
  4. The workbook contains zero or more named ranges or sheets that begin with the prefix “output_”. The data in these named ranges will be exposed to clients, such as AI models, for retrieval. Note that care 
  should be taken when setting the size of the named range.  It is recommended to use dynamic named ranges so the size of the named range can adapt to the size of the input dataset.  Also, it is optional but recommended that you assign data types to each column of the named range.

The user is free to write the business logic formulas in any way they choose.  The above standards only apply to the spreadsheet’s metadata, error-checking logic, inputs, and outputs.

Note that there should be no data in these workbooks unless they are constants.  All input named ranges should be empty.  The workbook should only contain formulas.

## Going Forward and Getting Involved

I’ll also note that there are some formulas that Apache POI does not currently support, such as Microsoft Excel’s FILTER formula.  In these more advanced uses cases, we can investigate adding formulas to Apache POI or 
another idea that we may provide additional advantages:  instead of a JAR wrapping an XLSX file, what if it were to wrap an in-memory SQLite database? Business "power users" could provide a SQL script using the same 
standards (e.g. tables prefixed with `input_` and `output_`). This gives them access to advanced relational logic, like the WHERE clause—which provides the same functionality as Excel’s FILTER formula—while maintaining 
the ability to download the database for local debugging. SQLite tables can also handle data in excess of XLSX’s limit of 1,048,576 while still allowing users to debug them locally.  This path ensures that as a user's 
business logic grows in complexity, the architecture can grow with them.  

If these ideas intrigue you, please contact me to get involved – whether that’s to share feedback, help build, user test, or something else.  The project has reached a point where I need other people’s perspectives on 
if and how this could be useful to others.

Thanks for reading!

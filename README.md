This is a workaround for unit testing a ASP.NET MVC 5 project that used ReactJS.NET with chutzpah in Visual Studio 2012.

This is the structure of the Visual Studio solution
```
ReactJS.NET Test
    |--Web (A ASP.NET MVC project)
    |--WebJSTests (A project to test the scripts in Web project)
```

#### Install React to the ASP.NET MVC 5 Project
1. Install "ReactJS.NET (MVC4 and 5)" (React.Web.Mvc4) via NuGet 
2. Install "react.js" via NuGet

#### Setup Unit Test Project
1. Create a Class Library project
2. Install "ReactJS.NET - Babel for MSBuild" (React.MSBuild) via NuGet
3. Add a properly configured `chutzpah.json` file in the project
4.  Comment or delete if the code below show up in the ASP.NET MVC project file

```
<Target Name="TransformBabel" AfterTargets="Build">
  <Exec Command="&quot;$(msbuildtoolspath)\msbuild.exe&quot; $(ProjectDirectory)TransformBabel.proj /p:OutputPath=$(OutputPath) /nr:false" />
</Target>
```

5. Edit the .csproj file of the test project, and add the following snippet before `</Project>`

```
<UsingTask AssemblyFile="$(OutputPath)\React.MSBuild.dll" TaskName="TransformBabel" />
<Target Name="TransformBabel" BeforeTargets="Build">
  <TransformBabel SourceDir="$(MSBuildProjectDirectory)\..\Web\Scripts" />
  <TransformBabel SourceDir="$(ProjectDir)" />
  <Exec Command="(robocopy &quot;$(ProjectDir)\..\Web\Scripts&quot; &quot;$(ProjectDir)\bin\generated&quot; &quot;*.generated.*&quot; /IS /IT /S /MOV /ndl /njh /njs /nc /ns /np) ^&amp; IF %ERRORLEVEL% LEQ 8 SET ERRORLEVEL=0" />
</Target>
```

7. Effectively, when you build `WebJSTests` project, it will transpile all the .jsx file in both `Web/Scripts` and `WebJSTests` folder and move those transpiled scripts in `Web/Scripts` to `WebJSTests/bin/generated`folder with `robocopy`.
8. When writing unit test in JSX, remember to build the project to tranpile.
9. When adding a new JSX test file in `WebJSTests` project, remember to add the transpiled .js file to the solution after build, so that the chutzpah can recognize the test in test runner.
10. You can reference the transpiled scripts from the test script in `WebJSTests` project. For example: `/// <reference path="./bin/generated/react-components/example.generated.js"/>`


Note: this unit test problem with ReactJS.NET could potentially solve more easily by using npm.

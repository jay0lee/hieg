<html>
<head>
<title>Header Insertion Extension Generator</title>

<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
<script src="jszip.min.js"></script>
<script src="FileSaver.min.js"></script>
<script>
function handleClick() {
  $.getJSON('manifest_template.json', function(manifest) {
    $.getJSON('rule_template.json', function(rule) {
      var ext_name = document.getElementById('ext_name').value;
      var header_name = document.getElementById('header_name').value;
      var header_value = document.getElementById('header_value').value;
      var host_patterns = document.getElementById('host_patterns').value.split('\n');
      manifest.name = ext_name;
      var nowd = new Date();
      var year = nowd.getUTCFullYear().toString();
      var month = (nowd.getUTCMonth() + 1).toString().padStart(2, "0");
      var dom = nowd.getUTCDate().toString().padStart(2, "0");
      var hour = nowd.getUTCHours().toString().padStart(2, "0");
      var minutes = nowd.getUTCMinutes().toString().padStart(2, "0");
      var seconds = nowd.getUTCSeconds().toString().padStart(2, "0");
      var ver_str = `${year}.${month}${dom}.${hour}.${minutes}${seconds}`;
      manifest.version = ver_str;

      // Set host permissions for extension manifest
      manifest.host_permissions = host_patterns

      // Set header value in rule template
      rule[0].action.requestHeaders[0].header = header_name
      rule[0].action.requestHeaders[0].value = header_value;

      // Build the zip file we'll "download"
      var zip = new JSZip();
      zip.file("rules1.json", JSON.stringify(rule, null, 2));
      zip.file("manifest.json", JSON.stringify(manifest, null, 2));
      zip.generateAsync({type:"blob"}).then(function(content) {
        saveAs(content, "org-restriction-" + manifest.version + ".zip");
      });
    });
  });
}

</script>
</head>
<body>
  <h2>Header Insertion Extension Generator</h2>
  This website will generate a Google Chrome extension which will add the specified header to URLs matching the specified patterns. Make sure your host patterns follow <a href="https://developer.chrome.com/docs/extensions/develop/concepts/match-patterns">the specified format</a>. 
<br><br>
<form name="exdetails" method="post" onSubmit="handleClick(); return false">
        Name your extension: <input type="text" id="ext_name" name="ext_name"><br>
	List <a href="https://developer.chrome.com/docs/extensions/develop/concepts/match-patterns">Host Patterns</a> to match: <br>
        <textarea id="host_patterns" name="host_patterns" rows="10" cols="15"></textarea><br>
       Header name: <input type="text" id="header_name" name="header_name" size="15"> Header value: <input type="text" id="header_value" name="header_value" size="25">
       <br>
        <input name="Submit"  type="submit" value="Generate Extension" />
</form>

  <h3>Quick test for your extension</h3>
  <ol>
    <li>Download and unzip your generated extension.</li>
  <li><a href="https://developer.chrome.com/docs/extensions/mv3/getstarted/development-basics/#load-unpacked">Install the unpacked extension</a></li>
  <li>You should now be restricted to using the GCP organizations you allowlisted when generating your extension.</li>
  </ol>

    <h3>Force installing the extension for users.</h3>
  <ol>
    <li>Download the extension and leave it as a .zip file.</li>
    <li>Upload the extension to the <a href="https://chrome.google.com/webstore/devconsole/">Chrome Web Store Developer Dashboard</a>.</li>
    <li>Publish the extension to your domain.</li>
    <li>In the admin console, force install the extension for your users.</li>
    <li>If you need to modify the list of allowlisted GCP organizations, generate a new extension with the list and then upload the new version to the CWS developer dashboard.</li>
  </ol>

</body>

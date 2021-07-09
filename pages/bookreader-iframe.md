---
title: Bookreader
permalink: /bookreader.html
---
<html>
<head>
    <title>bookreader demo</title>

    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
    <meta name="apple-mobile-web-app-capable" content="yes">

    <link rel="stylesheet" href="assets/lib/BookReader/BookReader.css"/>

    <script src="assets/lib/BookReader/jquery-1.10.1.js"></script>
    <script src="assets/lib/BookReader/jquery-ui-1.12.0.min.js"></script>
    <script src="assets/lib/BookReader/jquery.ui.touch-punch.min.js"></script>
    <script src="assets/lib/BookReader/jquery.browser.min.js"></script>
    <script src="assets/lib/BookReader/dragscrollable-br.js"></script>
    <script src="assets/lib/BookReader/jquery.colorbox-min.js"></script>

    <script src="assets/lib/BookReader/BookReader.js"></script>

    <!-- Plugins -->
    <script src="assets/lib/BookReader/plugins/plugin.iframe.js"></script>

    <style>
html, body, #BookReader { width: 100%; height:100%; margin:0; padding: 0; }
    </style>
</head>
<body style="background-color: #939598;">

<div id="BookReader">
  <noscript>
  <p>
      The BookReader requires JavaScript to be enabled. Please check that your browser supports JavaScript and that it is enabled in the browser settings.  You can also try one of the <a href="https://archive.org/details/goodytwoshoes00newyiala"> other formats of the book</a>.
  </p>
  </noscript>
</div>
<script type="text/javascript">
//Script adapted from Internet Archives iframe bookreader and script from OCLC CONTENTdm integration
	let json = {}	
    let server_url = '{{ site.cdm-url }}'
    let collection = '{{ site.cdm-collection-id }}'
    let item = window.location.href.split('#')[1]
 
let jsonData
    fetch(server_url + '//digital/iiif-info/' + collection + '/' + item + '/manifest.json')
    .then(response => {
        return response.json()
    }).then(data => {
        jsonData = data
    }).then(() => {
        let canvases = jsonData.sequences[0].canvases
        let imagesList = []
        let tempArray = []

        for(let i=0; i<canvases.length; i++)  {
            let imageObj = {}
            //Fullscreen
            let maxHeight = window.outerHeight
            let canvasHeight = canvases[i].height || 1200
            let canvasWidth = canvases[i].width || 800
            //Scale large images
            if(maxHeight < canvasHeight) {
                let scaleRatio = maxHeight / canvasHeight
                canvasHeight = Math.floor(canvasHeight*scaleRatio)
                canvasWidth = Math.floor(canvasWidth*scaleRatio)
            }

            let uri
            if(!canvases[i].images) {
                uri = `${window.location.origin}/iiif/2/${collection}:${item}/full/,${canvasHeight}/0/default.jpg?page=${i+1}`
            } else {
                uri = canvases[i].images[0].resource['@id']
                uri = uri.replace('/full/full/', `/full/,${canvasHeight}/`)
            }
            imageObj.width = canvasWidth
            imageObj.height = canvasHeight
            imageObj.uri = uri
            //Create an array(A) of arrays(B) where B would be the images you see on the page
            if(i === 0 || ((canvases.length % 2 === 0) && (i == canvases.length - 1))) {
                tempArray.push(imageObj)
                imagesList.push(tempArray)
                tempArray = []
            } else {
                tempArray.push(imageObj)
                if(tempArray.length == 2) {
                    imagesList.push(tempArray)
                    tempArray = []
                }
            }

        }
	
   //Some options we can pass into bookreader 
        let options = {
            data: imagesList,
			
  // Book title and the URL used for the book title link
    bookTitle: jsonData.label,
  //  bookUrl: '../index.html',
  //  bookUrlText: 'Back to item page',
    bookUrlTitle: jsonData.label,

    // thumbnail is optional, but it is used in the info dialog
    thumbnail: imagesList[0][0].uri,
    // Metadata is optional, but it is used in the info dialog
    metadata: jsonData.metadata,
  
  // Override the path used to find UI images
  imagesBaseURL: 'assets/lib/BookReader/images/',

  ui: 'responsive', // embed, full (responsive)

  el: '#BookReader',
  
  showToolbar: false
   

};

let br = new BookReader(options);
br.init();

}).catch(error => {
            console.log('Request failed: ' + error)
})

</script>

</body>
</html>
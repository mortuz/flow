<template>
  <v-container>
    <v-row class="mb-12">
      <v-col :cols="4">
        <v-textarea
          outlined
          v-model="flow"
          label="Editor"
          id="textarea"
          :auto-grow="true"
          @keyup="onFlowChange"
          @keydown="handleKeydown"
        ></v-textarea>
      </v-col>
      <v-col :cols="8" style="text-align: center">
        <div id="graph" v-html="diagram"></div>

        <section v-if="diagram">
          <v-btn :text="true" @click="onDownloadSVGClick">Download svg</v-btn>
          <v-btn :text="true" @click="onDownloadPNGClick">Download png</v-btn>
        </section>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
// @ is an alias to /src
HTMLTextAreaElement.prototype.getCaretPosition = function() {
  //return the caret position of the textarea
  return this.selectionStart;
};
HTMLTextAreaElement.prototype.setCaretPosition = function(position) {
  //change the caret position of the textarea
  this.selectionStart = position;
  this.selectionEnd = position;
  this.focus();
};
HTMLTextAreaElement.prototype.hasSelection = function() {
  //if the textarea has selection then return true
  if (this.selectionStart == this.selectionEnd) {
    return false;
  } else {
    return true;
  }
};
HTMLTextAreaElement.prototype.getSelectedText = function() {
  //return the selection text
  return this.value.substring(this.selectionStart, this.selectionEnd);
};
HTMLTextAreaElement.prototype.setSelection = function(start, end) {
  //change the selection area of the textarea
  this.selectionStart = start;
  this.selectionEnd = end;
  this.focus();
};

import * as mermaid from "mermaid";
mermaid.initialize({
  startOnLoad: true
});

export default {
  name: "Home",
  data: function() {
    return {
      flow: "",
      diagram: null
    };
  },
  methods: {
    onFlowChange: function(e) {
      try {
        // if (!this.flow) {
        //   this.diagram = null;
        // }
        mermaid.parse(this.flow);
        // let graphs = document.querySelectorAll("#dtheGraph");

        // if (graphs.length) {
        //   graphs[0].remove();
        // }

        let svg = null;
        mermaid.render("theGraph", this.flow, function(svgCode) {
          svg = svgCode;
        });

        if (svg) {
          this.diagram = svg;
        }
          console.log(': ----------------------------');
          console.log('this.diagram ', this.diagram );
          console.log(': ----------------------------');
      } catch (e) {
        console.log(e);
      }
    },

    handleKeydown: function(event) {
      let textarea = event.target;
      //support tab on textarea
      if (event.keyCode == 9) {
        //tab was pressed
        var newCaretPosition;
        newCaretPosition = textarea.getCaretPosition() + "    ".length;
        this.flow =
          this.flow.substring(0, textarea.getCaretPosition()) +
          "  " +
          this.flow.substring(textarea.getCaretPosition(), this.flow.length);
        textarea.setCaretPosition(newCaretPosition);
        event.preventDefault();
      }
      if (event.keyCode == 8) {
        //backspace
        if (
          this.flow.substring(
            textarea.getCaretPosition() - 4,
            textarea.getCaretPosition()
          ) == "  "
        ) {
          //it's a tab space
          var newCaretPosition;
          newCaretPosition = textarea.getCaretPosition() - 3;
          this.flow =
            this.flow.substring(0, textarea.getCaretPosition() - 3) +
            this.flow.substring(textarea.getCaretPosition(), this.flow.length);
          textarea.setCaretPosition(newCaretPosition);
        }
      }
      if (event.keyCode == 37) {
        //left arrow
        var newCaretPosition;
        if (
          this.flow.substring(
            textarea.getCaretPosition() - 4,
            textarea.getCaretPosition()
          ) == "  "
        ) {
          //it's a tab space
          newCaretPosition = textarea.getCaretPosition() - 3;
          textarea.setCaretPosition(newCaretPosition);
        }
      }
      if (event.keyCode == 39) {
        //right arrow
        var newCaretPosition;
        if (
          this.flow.substring(
            textarea.getCaretPosition() + 4,
            textarea.getCaretPosition()
          ) == "  "
        ) {
          //it's a tab space
          newCaretPosition = textarea.getCaretPosition() + 3;
          textarea.setCaretPosition(newCaretPosition);
        }
      }
    },
    onDownloadSVGClick: function(e) {
      e.preventDefault();

      var svgBlob = new Blob([this.diagram], {
        type: "image/svg+xml;charset=utf-8"
      });
      var svgUrl = URL.createObjectURL(svgBlob);
      var downloadLink = document.createElement("a");
      downloadLink.href = svgUrl;
      downloadLink.download = "graph.svg";
      document.body.appendChild(downloadLink);
      downloadLink.click();
      document.body.removeChild(downloadLink);
    },
    onDownloadPNGClick: function(e) {
      e.preventDefault();

      var wrapper = document.getElementById("graph");
      var svg = wrapper.querySelector("svg");

      if (typeof window.XMLSerializer != "undefined") {
        var svgData = new XMLSerializer().serializeToString(svg);
      } else if (typeof svg.xml != "undefined") {
        var svgData = svg.xml;
      }

      var canvas = document.createElement("canvas");
      var svgSize = svg.getBoundingClientRect();
      canvas.width = svgSize.width;
      canvas.height = svgSize.height;
      var ctx = canvas.getContext("2d");
      ctx.fillStyle = "#ffffff";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      var img = document.createElement("img");
      img.setAttribute(
        "src",
        "data:image/svg+xml;base64," +
          btoa(unescape(encodeURIComponent(svgData)))
      );

      img.onload = function() {
        ctx.drawImage(img, 0, 0);
        var imgsrc = canvas.toDataURL("image/png");

        var a = document.createElement("a");
        a.download = "diagram.png";
        a.href = imgsrc;
        a.click();
        document.body.removeChild(a);
        document.body.removeChild(img);
        document.body.removeChild(canvas);
      };
    }
  }
};
</script>

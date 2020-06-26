<template>
  <div>
    <div ref="viewer"></div>
    <template v-for="annot in annotations">
      <div class="dialog" :id="`dialog-${annot.id}`" :key="annot.id">
        <div
          class="dialog-anchor"
          :id="`dialog-${annot.id}-anchor`"
          slot="header"
        >
          <span>Click here to drag</span>
          <span class="clickable" @click="toggleDialog(`dialog-${annot.id}`)">
            X
          </span>
        </div>
        <!-- <el-main> -->
        <p>{{ annot }}</p>
        <!-- </el-main> -->
      </div>
    </template>
  </div>
</template>

<script>
/* eslint-disable */
import WebViewer from "@pdftron/webviewer";

export default {
  name: "WebViewer",
  props: {
    path: String,
    url: String,
  },
  data: function() {
    return {
      annotations: [
        { page: 1, x: 50, y: 50, w: 50, h: 20, id: "1" },
        { page: 1, x: 80, y: 90, w: 50, h: 20, id: "2" },
        { page: 1, x: 120, y: 120, w: 50, h: 20, id: "3" },
      ],
    };
  },
  methods: {
    toggleDialog: function(dialogId) {
      let dialogBox = document.querySelector(`#${dialogId}`);
      if (dialogBox.style.display === "block") dialogBox.style.display = "none";
      else dialogBox.style.display = "block";
    },
    makeElementDraggable: function(elmnt) {
      function dragMouseDown(e) {
        e = e || window.event;
        e.preventDefault();
        // get the mouse cursor position at startup:
        pos3 = e.clientX;
        pos4 = e.clientY;
        document.onmouseup = closeDragElement;
        // call a function whenever the cursor moves:
        document.onmousemove = elementDrag;
      }

      function elementDrag(e) {
        e = e || window.event;
        e.preventDefault();
        // calculate the new cursor 100position:
        pos1 = pos3 - e.clientX;
        pos2 = pos4 - e.clientY;
        pos3 = e.clientX;
        pos4 = e.clientY;
        // set the element's new position:
        elmnt.style.top = elmnt.offsetTop - pos2 + "px";
        elmnt.style.left = elmnt.offsetLeft - pos1 + "px";
      }

      function closeDragElement() {
        // stop moving when mouse button is released:
        document.onmouseup = null;
        document.onmousemove = null;
      }

      var pos1 = 0,
        pos2 = 0,
        pos3 = 0,
        pos4 = 0;
      if (document.getElementById(elmnt.id + "-anchor")) {
        // if present, the header is where you move the DIV from:
        document.getElementById(
          elmnt.id + "-anchor"
        ).onmousedown = dragMouseDown;
      } else {
        // otherwise, move the DIV from anywhere inside the DIV:
        elmnt.onmousedown = dragMouseDown;
      }
    },
  },
  created: function() {},
  mounted: function() {
    WebViewer(
      {
        path: this.path,
        initialDoc: this.url, // replace with your own PDF file
      },
      this.$refs.viewer
    ).then((instance) => {
      const { docViewer, annotManager, Annotations } = instance;

      instance.disableElements(["searchButton"]);
      instance.setTheme("dark");
      docViewer.setMargin(20);

      instance.disableElements(["annotationPopup", "contextMenuPopup"]);

      docViewer.on("documentLoaded", () => {
        for (let i = 0; i < this.annotations.length; i++) {
          let annot = this.annotations[i];

          const rectangleAnnot = new Annotations.RectangleAnnotation();
          rectangleAnnot.PageNumber = annot.page;
          rectangleAnnot.X = annot.x;
          rectangleAnnot.Y = annot.y;
          rectangleAnnot.Width = annot.w;
          rectangleAnnot.Height = annot.h;
          rectangleAnnot.Author = annotManager.getCurrentUser();
          rectangleAnnot.FillColor = new Annotations.Color(200, 0, 0, 0.4);
          rectangleAnnot.setCustomData("id", annot.id);

          annotManager.addAnnotation(rectangleAnnot);
          annotManager.redrawAnnotation(rectangleAnnot);
        }
      });

      annotManager.on("annotationSelected", (annotations, action) => {
        if (action == "selected") {
          let annotation = annotations[annotations.length - 1];
          let id = annotation.getCustomData("id");
          this.toggleDialog(`dialog-${id}`);
        }
      });

      docViewer.on("fitModeUpdated", (fitMode) => {
        console.log("fit mode changed");
      });

      instance.contextMenuPopup.add({
        type: "actionButton",
        label: "some-label",
        onClick: () => console.log("clicked"),
      });
    });

    for (let i = 0; i < this.annotations.length; i++) {
      let annot = this.annotations[i];
      let dialogBox = document.querySelector(`#dialog-${annot.id}`);
      // console.log(dialogBox);
      this.makeElementDraggable(dialogBox);
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div {
  width: 100%;
  height: 100vh;
}

.dialog {
  position: absolute;
  display: none;
  top: 50%;
  left: 50%;
  width: 10em;
  height: 10em;
  z-index: 50;
  background-color: white;
  border: 1px solid black;
}

.dialog-anchor {
  height: auto;
  background-color: lightslategray;
  cursor: move;
  display: flex;
  justify-content: space-around;
}

.clickable {
  cursor: pointer;
}
</style>

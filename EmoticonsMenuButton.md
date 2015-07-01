It was not so simple to figure out the best way for adding an emoticons menubutton in goog editor, so I am suggesting it here:

```
            // Create an editable field.
            var myField = new goog.editor.Field('editMe');

            // Create and register all of the editing plugins you want to use.
            myField.registerPlugin(new goog.editor.plugins.BasicTextFormatter());
            myField.registerPlugin(new goog.editor.plugins.ListTabHandler());
            myField.registerPlugin(new goog.editor.plugins.SpacesTabHandler());
            myField.registerPlugin(new goog.editor.plugins.EnterHandler());
            myField.registerPlugin(new goog.editor.plugins.LinkDialogPlugin());
            myField.registerPlugin(new goog.editor.plugins.LinkBubble());

           // Specify the buttons to add to the toolbar, using built in default buttons.
           var buttons = [
               goog.editor.Command.BOLD,
               goog.editor.Command.ITALIC,
               goog.editor.Command.UNDERLINE,
               goog.editor.Command.FONT_COLOR,
               goog.editor.Command.BACKGROUND_COLOR,
               goog.editor.Command.FONT_FACE,
               goog.editor.Command.FONT_SIZE,
               goog.editor.Command.UNORDERED_LIST,
               goog.editor.Command.ORDERED_LIST,
               goog.editor.Command.INDENT,
               goog.editor.Command.OUTDENT,
               goog.editor.Command.JUSTIFY_LEFT,
               goog.editor.Command.JUSTIFY_CENTER,
               goog.editor.Command.JUSTIFY_RIGHT,
               goog.editor.Command.JUSTIFY_FULL,
               goog.editor.Command.LINK
           ];
       
           var myToolbar = goog.ui.editor.DefaultToolbar.makeToolbar(buttons,goog.dom.getElement('toolbar'));

           var emoticons = [
              ['images/1.gif', 'std.200'],
              ['images/2.gif', 'std.201'],
              ['images/3.gif', 'std.202'],
              ['images/4.gif', 'std.203'],
              ['images/5.gif', 'std.204'],
              ['images/6.gif', 'std.205'],
              ['images/7.gif', 'std.206'],
              ['images/8.gif', 'std.2BC'],
              ['images/44.gif', 'std.2BD'],
              ['images/100.gif', 'std.2BE'],
              ['images/43.gif', 'std.2BF'],
              ['images/101.gif', 'std.2C0'],
              ['images/42.gif', 'std.2C1'],
              ['images/27.gif', 'std.2C2'],
              ['images/41.gif', 'std.2C3'],
              ['images/26.gif', 'std.2C4'],
              ['images/40.gif', 'std.2C5'],
              ['images/25.gif', 'std.2C6'],
              ['images/39.gif', 'std.2C7'],
              ['images/24.gif', 'std.2C8'],
              ['images/38.gif', 'std.2C9'],
              ['images/23.gif', 'std.2CA'],
              ['images/37.gif', 'std.2CB'],
              ['images/22.gif', 'std.2CC'],
              ['images/49.gif', 'std.2CD']
           ];
           var emojimenu = new goog.ui.Menu();
           var emojipalette=new goog.ui.emoji.EmojiPalette(emoticons);
           emojimenu.addChild(emojipalette,true)
          
           goog.events.listen(emojipalette, goog.ui.Component.EventType.ACTION, function(e){
               var field=myField;
               field.dispatchBeforeChange();
               var emoji = this.getSelectedEmoji();

  	           var image  = field.getEditableDomHelper().createElement(goog.dom.TagName.IMG);
  	           image.src = emoji.getUrl();
  	            
          	   field.focus();
          	   
          	   var range = field.getRange();
          	   
          	   image = range.replaceContentsWithNode(image);
  	
          	   // Done making changes, notify the editor.
          	   field.dispatchChange();

          	   // Put the user's selection right after the newly inserted image.
          	   goog.editor.range.placeCursorNextTo(image, false);

          	   // Dispatch selection change event since we just moved the selection.
          	   field.dispatchSelectionChangeEvent();
          	   emojimenu.setVisible(false);
      	           e.stopPropagation();
           });
           var node=goog.dom.createDom('div', {'class': 'tr-icon smile'});
           var nbutton=new goog.ui.ToolbarMenuButton(node,emojimenu);
           myToolbar.addChildAt(nbutton,15,true);
           // Hook the toolbar into the field.
           var myToolbarController = new goog.ui.editor.ToolbarController(myField, myToolbar);
           myField.makeEditable();
           myField.addListener([goog.events.EventType.CLICK,goog.events.EventType.KEYDOWN],function({
               myToolbarController.blur();
           });//hide menus when focusing field
```

Cheers
Matija
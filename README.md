inline 

html;

<div 
  [style.background-color]="backgroundColor"
  [style.width.px]="width"
  style="height: 100px;">
  <h2>Styled Div</h2>
  <button (click)="changeBackgroundColor()">Change Background Color</button>
  <button (click)="increaseWidth()">Increase Width</button>
</div>


ts;

import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app';
  backgroundColor: string = 'yellow';
  width: number = 100; // Initialize the width property

  changeBackgroundColor() {
    this.backgroundColor =
      this.backgroundColor === 'yellow' ? 'lightblue' : 'yellow';
  }

  increaseWidth() {
    this.width += 20; // Increment the width by 20 pixels
  }
}

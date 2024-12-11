import { Directive, ElementRef, HostListener, Input, forwardRef } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';
import BigNumber from 'bignumber.js';

@Directive({
  selector: '[appCurrencyInput]',
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => CurrencyInputDirective), // Tránh vòng lặp
      multi: true,
    },
  ],
})
export class CurrencyInputDirective implements ControlValueAccessor {
  @Input() decimalPlaces: number = 2; // Số thập phân
  @Input() currencySymbol: string = '$'; // Ký hiệu tiền tệ

  private rawValue: string = ''; // Giá trị thực tế không định dạng
  private onChange: (value: any) => void = () => {};
  private onTouched: () => void = () => {};

  constructor(private el: ElementRef) {}

  @HostListener('input', ['$event.target.value'])
  onInput(value: string): void {
    // Xử lý khi người dùng nhập dữ liệu vào
    this.rawValue = this.unformatCurrency(value);
    this.el.nativeElement.value = this.rawValue; // Hiển thị giá trị chưa định dạng khi nhập
    this.onChange(this.rawValue); // Gửi giá trị thực tế cho FormControl
  }

  @HostListener('blur')
  onBlur(): void {
    // Khi mất focus, hiển thị giá trị đã định dạng
    this.el.nativeElement.value = this.formatCurrency(this.rawValue);
    this.onTouched();
  }

  @HostListener('focus')
  onFocus(): void {
    // Khi focus vào input, hiển thị giá trị thực tế
    this.el.nativeElement.value = this.rawValue;
  }

  writeValue(value: any): void {
    // Khi giá trị được gán cho form control từ bên ngoài
    this.rawValue = value || '';
    this.el.nativeElement.value = this.formatCurrency(this.rawValue);
  }

  registerOnChange(fn: any): void {
    this.onChange = fn;
  }

  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }

  private formatCurrency(value: string | number): string {
    if (!value) return '';

    const bigNumberValue = new BigNumber(value);
    const formattedValue = bigNumberValue.toFixed(this.decimalPlaces);

    const parts = formattedValue.split('.');
    let integerPart = parts[0];
    const decimalPart = parts[1] || '00';
    integerPart = integerPart.replace(/\B(?=(\d{3})+(?!\d))/g, ','); // Thêm dấu phân cách hàng nghìn

    return `${this.currencySymbol}${integerPart}.${decimalPart}`;
  }

  private unformatCurrency(value: string): string {
    // Loại bỏ các ký tự không hợp lệ
    return value.replace(/[^\d.-]/g, '');
  }
}

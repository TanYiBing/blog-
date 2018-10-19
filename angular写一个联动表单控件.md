---
title: angular写一个联动表单控件
date: 2018-10-18 19:48:48
tags:
- Angular
- Typescript
- Material
categories:
- Angular
---
首先展示下这个表单的样式：

>![](/img/form/1.png)

这就是我们需要设计成的表单样式，从图中我们可以看出这个表单可以分成三个部分：

1. 证件输入部分
2. 出生日期部分
3. 地址部分

我们不应该将这些部分都写在一起，因为这样我们的表单将会特别复杂，因为我们将所有的逻辑都写在了一起，而且也不适合复用。所有我们将这三个部分写成三个控件，下面就分别给出模版：

1. 证件输入部分模版：

		<div>
			<mat-form-field>
				<mat-select placeholder="证件类型" (ngModelChange)="onIdTypeChange($event.value)" [(ngModel)]="identity.identityType">
					<mat-option *ngFor="let type of identityTypes" [value]="type.value">
	                {{type.label}}
					</mat-option>
				</mat-select>
			</mat-form-field>
		</div>
		<div class="id-input">
	    	<mat-form-field>
	        	<input matInput type="text" placeholder="证件号码" (change)="onIdNoChange($event.target.value)" [(ngModel)]="identity.identityNo">
	        <mat-error>证件号码输入有误</mat-error>
	    	</mat-form-field>
		</div>

2. 出生日期部分模版:

		<div [formGroup]="form">
			<div>
			    <mat-form-field>
					<input matInput [matDatepicker]="birthPicker" type="text" placeholder="出生日期" formControlName="birthday">
					<mat-datepicker-toggle matSuffix [for]="birthPicker"></mat-datepicker-toggle>
					<mat-datepicker #dueDatePicker></mat-datepicker>
					<mat-error>日期不正确</mat-error>
			    </mat-form-field>
			    <mat-datepicker touchUi="true" #birthPicker></mat-datepicker>
			</div>
			<ng-container formGroupName="age">
			    <div>
					<mat-form-field>
						<input matInput type="number" placeholder="年龄" formControlName="ageNum">
					</mat-form-field>
			    </div>
			    <div>
					<mat-button-toggle-group formControlName="ageUnit" [(ngModel)]="selectedUnit">
						<mat-button-toggle *ngFor="let unit of ageUnits" [value]="unit.value">
							{{ unit.label }}
						</mat-button-toggle>
					</mat-button-toggle-group>
			    </div>
			    <mat-error class="mat-body-2" *ngIf="form.get('age').hasError('ageInvalid')">年龄或单位不正确</mat-error>
			</ng-container>
		</div>


3. 地址部分模版:

		<div class="address-group">
		    <mat-form-field>
		        <mat-select placeholder="请选择省份" [(ngModel)]="_address.province" (ngModelChange)="onProvinceChange()">
		            <mat-option *ngFor="let p of provinces$ | async" [value]="p">
		                {{ p }}
		            </mat-option>
		        </mat-select>
		    </mat-form-field>
		    <mat-form-field>
		        <mat-select placeholder="请选择城市" [(ngModel)]="_address.city" (ngModelChange)="onCityChange()">
		            <mat-option *ngFor="let c of cities$ | async" [value]="c">
		                {{ c }}
		            </mat-option>
		        </mat-select>
		    </mat-form-field>
		    <mat-form-field>
		        <mat-select placeholder="请选择区县" [(ngModel)]="_address.district" (ngModelChange)="onDistrictChange()">
		            <mat-option *ngFor="let d of districts$ | async" [value]="d">
		                {{ d }}
		            </mat-option>
		        </mat-select>
		    </mat-form-field>
		    <div class="street">
		        <mat-form-field>
		            <input matInput placeholder="街道地址" [(ngModel)]="_address.street" (ngModelChange)="onStreetChange()">
		        </mat-form-field>
		    </div>
		</div>

我使用的是`material`主题，模版看起来可能有点复杂。接下来我们分别看下每个模块需要实现的效果：

1. 证件部分就是正常的输入，但是后面要实现一个联动的效果，因为省份证包含了很多信息。

	>![](/img/form/shenfen.gif)

2. 年龄部分本身就需要一个联动，效果如下：

	>![](/img/form/age.gif)

3. 地区部分需要需要根据其他选择项进行筛选：

	>![](/img/form/area.gif)

4. 最后我们需要有一个根据身份证信息，得到我们地址和生日的联动，最终的效果如下：

	>![](/img/form/form.gif)

效果我们有已经看到了，接下来就要对各个控件进行编写了：

1. 证件部分:

		import { Component, OnInit, forwardRef, OnDestroy, ChangeDetectionStrategy } from '@angular/core';
		import { NG_VALUE_ACCESSOR, NG_VALIDATORS, ControlValueAccessor, FormControl } from '@angular/forms';
		import { Subscription, Subject, Observable, combineLatest } from 'rxjs';
		import { IdentityType, Identity } from '../../domain';
		
		@Component({
		    selector: 'app-identity-input',
		    templateUrl: './identity-input.component.html',
		    styleUrls: ['./identity-input.component.scss'],
		    providers: [
		        {
		            provide: NG_VALUE_ACCESSOR,
		            useExisting: forwardRef(() => IdentityInputComponent),
		            multi: true,
		        },
		        {
		            provide: NG_VALIDATORS,
		            useExisting: forwardRef(() => IdentityInputComponent),
		            multi: true,
		        }
		    ],
		    changeDetection: ChangeDetectionStrategy.OnPush,
		})
		export class IdentityInputComponent implements ControlValueAccessor, OnInit, OnDestroy {
		
		    identityTypes: { value: IdentityType, label: string }[] = [
		        { value: IdentityType.IdCard, label: '身份证' },
		        { value: IdentityType.Insurance, label: '医保' },
		        { value: IdentityType.Passport, label: '护照' },
		        { value: IdentityType.Military, label: '军官证' },
		        { value: IdentityType.Other, label: '其它' }
		    ];
		    identity: Identity = { identityType: null, identityNo: null };
		    private _idType = new Subject<IdentityType>();
		    private _idNo = new Subject<string>();
		    private _sub: Subscription;
		    private propagateChange = (_: any) => { };
		
		    constructor() { }
		
		    ngOnInit() {
		        const val$ = combineLatest(this.idType, this.idNo, (_type, _no) => {
		            return {
		                identityType: _type,
		                identityNo: _no
		            };
		        });
		        this._sub = val$.subscribe(v => {
		            this.identity = v;
		            this.propagateChange(v);
		        });
		    }
		
		    ngOnDestroy(): void {
		        if (this._sub) {
		            this._sub.unsubscribe();
		        }
		    }
		
		    writeValue(obj: any): void {
		    }
		
		    registerOnChange(fn: any): void {
		        this.propagateChange = fn;
		    }
		
		    registerOnTouched(fn: any): void {
		    }
		
		    setDisabledState?(isDisabled: boolean): void {
		    }
		
		    // 验证表单，验证结果正确返回 null 否则返回一个验证结果对象
		    validate(c: FormControl): { [key: string]: any } {
		        if (!c.value) {
		            return null;
		        }
		        switch (c.value.identityType) {
		            case IdentityType.IdCard: {
		                return this.validateIdCard(c);
		            }
		            case IdentityType.Passport: {
		                return this.validatePassport(c);
		            }
		            case IdentityType.Military: {
		                return this.validateMilitary(c);
		            }
		            case IdentityType.Insurance:
		            default: {
		                return null;
		            }
		        }
		    }
		
		    onIdTypeChange(idType) {
		        this._idType.next(idType);
		    }
		
		    onIdNoChange(idNo) {
		        this._idNo.next(idNo);
		    }
		
		    private get idType(): Observable<IdentityType> {
		        return this._idType.asObservable();
		    }
		
		    private get idNo(): Observable<string> {
		        return this._idNo.asObservable();
		    }
		
		    private validateIdCard(c: FormControl): { [key: string]: any } {
		        const val = c.value.identityNo;
		        if (val.length !== 18) {
		            return {
		                idNotValid: true
		            };
		        }
		        const pattern = /^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}[x0-9]$/;
		        return pattern.test(val) ? null : { idNotValid: true };
		    }
		
		    private validatePassport(c: FormControl): { [key: string]: any } {
		        const value = c.value.identityNo;
		        if (value.length !== 9) {
		            return { idNotValid: true };
		        }
		        const pattern = /^[GgEe]\d{8}$/;
		        return pattern.test(value) ? null : { idNotValid: true };
		    }
		
		    private validateMilitary(c: FormControl): { [key: string]: any } {
		        const value = c.value.identityNo;
		        const pattern = /[\u4e00-\u9fa5](字第)(\d{4,8})(号?)$/;
		        return pattern.test(value) ? null : { idNotValid: true };
		    }
		}

2. 年龄部分:

		import { Component, forwardRef, OnInit, OnDestroy, Input } from '@angular/core';
		import { FormGroup, NG_VALUE_ACCESSOR, NG_VALIDATORS, ControlValueAccessor, FormControl, FormBuilder } from '@angular/forms';
		import { map, merge, filter, startWith, debounceTime, distinctUntilChanged } from 'rxjs/operators';
		import { Observable, Subscription, combineLatest } from 'rxjs';
		import {
		    subDays,
		    subMonths,
		    subYears,
		    differenceInDays,
		    differenceInMonths,
		    differenceInYears,
		    isBefore,
		    parse,
		    format
		} from 'date-fns';
		import { isValidDate } from '../../utils/date.util';
		
		export enum AgeUnit {
		    Year = 0,
		    Month,
		    Day
		}
		
		export interface Age {
		    age: number;
		    unit: AgeUnit;
		}
		
		@Component({
		    selector: 'app-age-input',
		    templateUrl: './age-input.component.html',
		    styleUrls: ['./age-input.component.scss'],
		    providers: [
		        {
		            provide: NG_VALUE_ACCESSOR,
		            useExisting: forwardRef(() => AgeInputComponent),
		            multi: true // 允许令牌多对一
		        },
		        {
		            provide: NG_VALIDATORS,
		            useExisting: forwardRef(() => AgeInputComponent),
		            multi: true
		        }
		    ]
		})
		export class AgeInputComponent implements ControlValueAccessor, OnInit, OnDestroy {
		
		    @Input() daysTop = 90;
		    @Input() daysBottom = 0;
		    @Input() monthsTop = 24;
		    @Input() monthsBottom = 1;
		    @Input() yearsTop = 150;
		    @Input() yearsBottom = 1;
		    @Input() format = 'YYYY-MM-DD';
		    @Input() debounceTime = 300;
		    selectedUnit = AgeUnit.Year;
		    ageUnits = [
		        { value: AgeUnit.Year, label: '岁' },
		        { value: AgeUnit.Month, label: '月' },
		        { value: AgeUnit.Day, label: '日' }
		    ];
		    form: FormGroup;
		    sub: Subscription;
		    private propagateChange = (_: any) => { };
		
		    constructor(private fb: FormBuilder) { }
		
		    ngOnInit(): void {
		        this.form = this.fb.group({
		            birthday: ['', this.validateDate],
		            age: this.fb.group({
		                ageNum: [''],
		                ageUnit: [AgeUnit.Year]
		            }, { validator: this.validateAge('ageNum', 'ageUnit') })
		        });
		        const birthday = this.form.get('birthday');
		        const ageNum = this.form.get('age').get('ageNum');
		        const ageUnit = this.form.get('age').get('ageUnit');
		
		        const birthday$ = birthday.valueChanges.pipe(
		            map(d => {
		                return { date: d, from: 'birthday' };
		            }),
		            debounceTime(this.debounceTime),
		            distinctUntilChanged(),
		            filter(_ => birthday.valid)
		        );
		        const ageNum$ = ageNum.valueChanges.pipe(
		            startWith(ageNum.value),
		            debounceTime(this.debounceTime),
		            distinctUntilChanged(),
		        );
		        const ageUnit$ = ageUnit.valueChanges.pipe(
		            startWith(ageUnit.value),
		            debounceTime(this.debounceTime),
		            distinctUntilChanged()
		        );
		
		        // rxjs6版本中方法被改造了
		        const age$ = combineLatest(ageNum$, ageUnit$).pipe(
		            map(([_n, _u]) => this.toDate({ age: _n, unit: _u })),
		            map(d => {
		                return { date: d, from: 'age' };
		            }),
		            filter(_ => this.form.get('age').valid)
		        );
		        const merged$ = Observable.prototype.pipe(
		            merge(birthday$, age$),
		            filter(_ => this.form.valid)
		        );
		        this.sub = merged$.subscribe(d => {
		            const age = this.toAge(d.date);
		            if (d.from === 'birthday') {
		                if (age.age !== ageNum.value) {
		                    ageNum.patchValue(age.age, { emitEvent: false });
		                }
		                if (age.unit !== ageUnit.value) {
		                    this.selectedUnit = age.unit;
		                    ageUnit.patchValue(age.age, { emitEvent: false });
		                }
		                this.propagateChange(d.date);
		            } else {
		                const ageToCompare = this.toAge(this.form.get('birthday').value);
		                if (age.age !== ageToCompare.age || age.unit !== ageToCompare.unit) {
		                    birthday.patchValue(d.date, { emitEvent: false });
		                    this.propagateChange(d.date);
		                }
		            }
		        });
		    }
		
		    ngOnDestroy(): void {
		        if (this.sub) {
		            this.sub.unsubscribe();
		        }
		    }
		
		    writeValue(obj: any): void {
		        if (obj) {
		            const date = format(obj, this.format);
		            this.form.get('birthday').patchValue(date);
		            const age = this.toAge(date);
		            this.form.get('age').get('ageNum').patchValue(age.age);
		            this.form.get('age').get('ageUnit').patchValue(age.unit);
		        }
		    }
		    registerOnChange(fn: any): void {
		        this.propagateChange = fn;
		    }
		    registerOnTouched(fn: any): void {
		    }
		    setDisabledState?(isDisabled: boolean): void {
		    }
		
		
		    toAge(dateStr: string): Age {
		        const date = parse(dateStr);
		        const now = Date.now();
		        return isBefore(subDays(now, this.daysTop), date) ?
		            { age: differenceInDays(now, date), unit: AgeUnit.Day } :
		            isBefore(subMonths(now, this.monthsTop), date) ?
		                { age: differenceInMonths(now, date), unit: AgeUnit.Month } :
		                { age: differenceInYears(now, date), unit: AgeUnit.Year };
		    }
		
		    toDate(age: Age): string {
		        const now = Date.now();
		        switch (age.unit) {
		            case AgeUnit.Year: {
		                return format(subYears(now, age.age), this.format);
		            }
		            case AgeUnit.Month: {
		                return format(subMonths(now, age.age), this.format);
		            }
		            case AgeUnit.Day: {
		                return format(subDays(now, age.age), this.format);
		            }
		            default: {
		                return null;
		            }
		        }
		    }
		
		    validate(c: FormControl): { [key: string]: any } {
		        const val = c.value;
		        if (!val) {
		            return null;
		        }
		        if (isValidDate(val)) {
		            return null;
		        }
		        return {
		            dateOfBirthInvalid: true
		        };
		    }
		
		
		    // 表单验证器
		    validateAge(ageNumkey: string, ageUnitKey: string) {
		        return (group: FormGroup): { [key: string]: any } => {
		            const ageNum = group.controls[ageNumkey];
		            const ageUnit = group.controls[ageUnitKey];
		            let result = false;
		            const ageNumVal = ageNum.value;
		            switch (ageUnit.value) {
		                case AgeUnit.Year: {
		                    result = ageNumVal >= this.yearsBottom && ageNumVal < this.yearsTop;
		                    break;
		                }
		                case AgeUnit.Month: {
		                    result = ageNumVal >= this.monthsBottom && ageNumVal < this.monthsTop;
		                    break;
		                }
		                case AgeUnit.Day: {
		                    result = ageNumVal >= this.daysBottom && ageNumVal < this.daysTop;
		                    break;
		                }
		                default: {
		                    break;
		                }
		            }
		            return result ? null : { ageInvalid: true };
		        };
		    }
		
		    validateDate(c: FormControl): { [key: string]: any } {
		        const val = c.value;
		        return isValidDate(val) ? null : {
		            birthdayInvalid: true
		        };
		    }
		
		}


3. 地区部分:

		import { Component, OnInit, forwardRef, ChangeDetectionStrategy, OnDestroy } from '@angular/core';
		import { NG_VALUE_ACCESSOR, NG_VALIDATORS, ControlValueAccessor, FormControl } from '@angular/forms';
		import { Subscription, Subject, combineLatest, Observable, of } from 'rxjs';
		import { Address } from '../../domain/user.model';
		import { startWith, map } from 'rxjs/operators';
		import { getProvinces, getCitiesByProvince, getAreasByCity } from '../../utils/area.util';
		
		@Component({
		    selector: 'app-area-list',
		    templateUrl: './area-list.component.html',
		    styleUrls: ['./area-list.component.scss'],
		    providers: [
		        {
		            provide: NG_VALUE_ACCESSOR,
		            useExisting: forwardRef(() => AreaListComponent),
		            multi: true,
		        },
		        {
		            provide: NG_VALIDATORS,
		            useExisting: forwardRef(() => AreaListComponent),
		            multi: true,
		        },
		    ],
		    changeDetection: ChangeDetectionStrategy.OnPush,
		})
		export class AreaListComponent implements ControlValueAccessor, OnInit, OnDestroy {
		
		    _address: Address = {
		        province: '',
		        city: '',
		        district: '',
		        street: ''
		    };
		    _province = new Subject();
		    _city = new Subject();
		    _district = new Subject();
		    _street = new Subject();
		    provinces$: Observable<string[]>;
		    cities$: Observable<string[]>;
		    districts$: Observable<string[]>;
		    private _sub: Subscription;
		    private propagateChange = (_: any) => { };
		
		    constructor() { }
		
		    ngOnInit() {
		        const province$ = this._province.asObservable().pipe(startWith(''));
		        const city$ = this._city.asObservable().pipe(startWith(''));
		        const district$ = this._district.asObservable().pipe(startWith(''));
		        const street$ = this._street.asObservable().pipe(startWith(''));
		        const val$ = combineLatest([province$, city$, district$, street$], (_p, _c, _d, _s) => {
		            return {
		                province: _p,
		                city: _c,
		                district: _d,
		                street: _s
		            };
		        });
		        this._sub = val$.subscribe(v => {
		            this.propagateChange(v);
		        });
		
		        this.provinces$ = of(getProvinces());
		        // 根据省份的选择得到城市列表
		        this.cities$ = province$.pipe(
		            map((p: string) => getCitiesByProvince(p))
		        );
		        // 根据省份和城市的选择得到地区列表
		        this.districts$ = combineLatest(province$, city$, (p, c) => ({ province: p, city: c })).pipe(
		            map(a => getAreasByCity(a.province, a.city))
		        );
		    }
		
		    ngOnDestroy(): void {
		        if (this._sub) {
		            this._sub.unsubscribe();
		        }
		    }
		
		    // 设置初始值
		    writeValue(obj: Address): void {
		        if (obj) {
		            this._address = obj;
		            if (this._address.province) {
		                this._province.next(this._address.province);
		            }
		            if (this._address.city) {
		                this._city.next(this._address.city);
		            }
		            if (this._address.district) {
		                this._district.next(this._address.district);
		            }
		            if (this._address.street) {
		                this._street.next(this._address.street);
		            }
		        }
		    }
		    registerOnChange(fn: any): void {
		        this.propagateChange = fn;
		    }
		    registerOnTouched(fn: any): void {
		    }
		    setDisabledState?(isDisabled: boolean): void {
		    }
		
		    // 验证表单，验证结果正确返回 null 否则返回一个验证结果对象
		    validate(c: FormControl): { [key: string]: any } {
		        const val = c.value;
		        if (!val) {
		            return null;
		        }
		        if (val.province && val.city && val.district && val.street && val.street.length >= 4) {
		            return null;
		        }
		        return {
		            addressNotValid: true
		        };
		    }
		
		    onProvinceChange() {
		        this._province.next(this._address.province);
		    }
		
		    onCityChange() {
		        this._city.next(this._address.city);
		    }
		
		    onDistrictChange() {
		        this._district.next(this._address.district);
		    }
		
		    onStreetChange() {
		        this._street.next(this._address.street);
		    }
		}

当然了，这些不是全部的代码，有些重复的功能我写成了单独的ts文件，有兴趣的可以去我的[github](https://github.com/TanYiBing/taskmgr)上看。

这样我们只是完成了各个自定义控件，这样的好处是我们的控件可以重复使用，而且各部分逻辑都在各自的部分，所以最后我们需要一个整体来包含这些控件。

	<form [formGroup]="form" (ngSubmit)="onSubmit(form, $event)">
	    <mat-card>
	        <mat-tab-group>
	            <mat-tab label="帐号信息">
	                <mat-card-header>
	                    <mat-card-title>注册</mat-card-title>
	                </mat-card-header>
	                <mat-card-content>
	                    <mat-form-field class="full-width">
	                        <input matInput placeholder="您的email*" type="email" formControlName="email">
	                    </mat-form-field>
	                    <mat-form-field class="full-width">
	                        <input matInput placeholder="您的姓名*" type="text" formControlName="name">
	                    </mat-form-field>
	                    <mat-form-field class="full-width">
	                        <input matInput placeholder="输入您的密码*" type="password" formControlName="password">
	                    </mat-form-field>
	                    <mat-form-field class="full-width">
	                        <input matInput placeholder="重复输入您的密码*" type="password" formControlName="repeat">
	                    </mat-form-field>
	                    <app-image-list-select [useSvgIcon]="true" [cols]="6" [title]="'选择头像：'" [items]="items" formControlName="avatar">
	                    </app-image-list-select>
	                    <button mat-raised-button color="primary" type="submit">注册</button>
	                </mat-card-content>
	                <mat-card-actions class="text-right">
	                    <p>已有账户？<a href="">登录</a></p>
	                    <p>忘记密码？<a href="">找回</a></p>
	                </mat-card-actions>
	            </mat-tab>
	            <mat-tab label="个人信息">
	                <div class="full-width control-padding">
	                    <app-identity-input formControlName="identity" class="full-width control-padding"></app-identity-input>
	                </div>
	                <div class="full-width control-padding">
	                    <app-age-input formControlName="dateOfBirth"></app-age-input>
	                </div>
	                <div class="full-width control-padding">
	                    <app-area-list formControlName="address"></app-area-list>
	                </div>
	            </mat-tab>
	        </mat-tab-group>
	    </mat-card>
	</form>

这个部分的处理逻辑如下：

	import { Component, OnInit, OnDestroy } from '@angular/core';
	import { FormBuilder, Validators, FormGroup } from '@angular/forms';
	import { debounceTime, filter } from 'rxjs/operators';
	import { Subscription } from 'rxjs';
	import { extractInfo, isValidAddr, getAddrByCode } from '../../utils/identity.util';
	import { isValidDate } from '../../utils/date.util';
	
	@Component({
	    selector: 'app-register',
	    templateUrl: './register.component.html',
	    styleUrls: ['./register.component.scss']
	})
	export class RegisterComponent implements OnInit, OnDestroy {
	
	    items: string[];
	    form: FormGroup;
	    sub: Subscription;
	    private readonly avatarName = 'avatars';
	    constructor(private fb: FormBuilder) { }
	
	    ngOnInit() {
	        const img = `${this.avatarName}:svg-${(Math.random() * 16).toFixed()}`;
	        const nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16];
	        this.items = nums.map(val => `avatars:svg-${val}`);
	        this.form = this.fb.group({
	            email: ['wang@163.com', Validators.compose([Validators.email, Validators.required])],
	            name: [''],
	            password: [''],
	            repeat: [''],
	            avatar: [img],
	            dateOfBirth: ['1990-01-01'],
	            address: [''],
	            identity: ['']
	        });
	        const id$ = this.form.get('identity').valueChanges.pipe(
	            debounceTime(300),
	            filter(_ => this.form.get('identity').valid)
	        );
	        this.sub = id$.subscribe(id => {
	            const info = extractInfo(id.identityNo);
	            if (isValidAddr(info.addrCode)) {
	                const addr = getAddrByCode(info.addrCode);
	                this.form.get('address').patchValue(addr);
	            }
	            if (isValidDate(info.dateOfBirth)) {
	                this.form.get('dateOfBirth').patchValue(info.dateOfBirth);
	            }
	        });
	    }
	
	    ngOnDestroy(): void {
	        if (this.sub) {
	            this.sub.unsubscribe();
	        }
	    }
	
	    onSubmit({ value, valid }, ev: Event) {
	        ev.preventDefault();
	        if (!valid) {
	            return;
	        }
	        console.log(value);
	    }
	}


这样就形成了一个完整的自定义表单控件，而且里面部分还可以拿出来单独使用。
# Redux-form example

## Let's discuss utilizing forms in React on sign-up form example

[https://redux-form.com/7.2.0/](https://redux-form.com/7.2.0/)

##
    
    * redux-form library observation
    * <Field/> component
    * reducer.plugin extension
    
### Data flow

![https://github.com/erikras/redux-form/raw/master/docs/reduxFormDiagram.png](https://github.com/erikras/redux-form/raw/master/docs/reduxFormDiagram.png)

### <Field/>

The <Field/> component connects each input to the store.

````
<Field
    name={fieldsNames.EMAIL}
    type="text"
    component={OBFormInput}
    props={{
      label: signUpMessages.labelEmail
    }}
    validate={[requiredField, emailField]}
/>
````

<OBFormInput/>:

````
const shouldDisplayError = ({ active, touched, invalid, submitting, error }) => {
  if (active) return false;
  if (invalid && submitting) return true;
  if (invalid && touched) return true;
  return false;
};

<div className={classNames(styles.wrapper, { [styles.hasErrors]: shouldDisplayError(meta) })}>
      <label htmlFor={name}>
        <FormattedMessage {...label} />
      </label>
      <div>
        <input
          {...input}
          type={type}
          placeholder={placeholder}
        />
        {shouldDisplayError(meta) && <span>{meta.error}</span>}
      </div>
    </div>
````

### Form example
````
handleSubmit = () => this.props.dispatch(signUpRequest()); // dispatching action creator

shouldViewCommonValidationError = !submitting && !valid && !_.isEmpty(error);

@reduxForm({
  form: signUpFormName, //describes form name in string
  initialValues // describes initial values
})
export default class SignUpForm extends PureComponent {
    <form>
            <Field
                name={fieldsNames.EMAIL}
                type="text"
                component={OBFormInput}
                props={{
                  label: signUpMessages.labelEmail
                }}
                validate={[requiredField, emailField]}
              />
              
            <OBSubmitButton
                       disabled={!valid || submitting}
                       loading={submitting}
                       onClick={this.handleSubmit}
             >
             
            {shouldViewCommonValidationError && (
                      <ErrorNotification>
                        {_.isString(error) ? error : _.get('message', error)}
                      </ErrorNotification>
                    )}
    </form>
}
````

### Sign up duck (saga)

````

export const SIGN_UP_REQUEST = 'SIGN_UP_REQUEST';
export const SIGN_UP_SUCCESS = 'SIGN_UP_SUCCESS';
export const SIGN_UP_FAILURE = 'SIGN_UP_FAILURE';

export const signUpRequest = createAction(SIGN_UP_REQUEST);
export const signUpSuccess = createAction(SIGN_UP_SUCCESS);
export const signUpFailure = createAction(SIGN_UP_FAILURE);

export function* signUpSaga({ payload }) {
  try {
    const storeFormValues = yield select(getFormValues(signUpFormName)); // let's take values from store 'form'
    const { password, ...userRegistrationData } = _.cond([
      [_.has('values.password'), _.get('values')], // maybe form values was in 'values' in action payload
      [_.has('password'), _.identity], // maybe action payload - is form values
      [_.stubTrue, _.constant(storeFormValues)] // looks like form values should be taken from store directly (default behaviour)
    ])(payload);
    const ambisafeAccountContainer = generateAmbisafeAccountContainer(password);
    const hashedPassword = hashPassword(password);
    const userRegistrationDataDTO = {
      password: hashedPassword,
      container: ambisafeAccountContainer,
      ...userRegistrationData
    };

    const userSignUpResponse = yield call(fetchAuth.submitUserRegistrationData, userRegistrationDataDTO);
    const isUserSignUpSuccess = _.flow(_.get('status'), _.eq('ok'))(userSignUpResponse);

    if (isUserSignUpSuccess) {
      return yield all([
        put(setAuthContainer(_.get('container', userSignUpResponse))),
        put(signUpSuccess({ password, ...userRegistrationData }))
      ]);
    }

    return put(signUpFailure(new Error('Unsuccessful registration')));
  } catch (e) {
    yield put(signUpFailure(e));
  }
}
````

### Additional reducer example

````
import { SIGN_UP_FAILURE, SIGN_UP_SUCCESS, SIGN_UP_REQUEST } from '../../../../ducks/auth/sign-up/sign-up.duck';
import signUpFormName from './sign-up-form.names';

export default function (state, { type, payload }) {
  switch (type) {
  case SIGN_UP_REQUEST:
    return {
      ...state,
      submitting: true,
    };
  case SIGN_UP_SUCCESS:
    return {
      ...state,
      submitting: false,
      submitFailed: false,
      submitSucceeded: true,
    };
  case SIGN_UP_FAILURE:
    return {
      ...state,
      error: payload,
      submitting: false,
      submitFailed: true,
      submitSucceeded: false,
    };
  default:
    return state;
  }
}
````

Root reducer:
````
const rootReducer = combineReducers({
  auth: authReducer,
  router: routerReducer,
  form: formReducer.plugin({
    [signUpFormName]: signUpFormAdditionalReducer
  }),
});
````
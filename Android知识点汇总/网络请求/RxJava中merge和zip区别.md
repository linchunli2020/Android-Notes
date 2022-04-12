***----------------------------------------zip-以小(发射器B)为主-----------------------------------------------***

      public static <T1, T2, R> Observable<R> zip(
              ObservableSource<? extends T1> source1, ObservableSource<? extends T2> source2,
              BiFunction<? super T1, ? super T2, ? extends R> zipper) {
          // ......
          return zipArray(Functions.toFunction(zipper), false, bufferSize(), source1, source2);
      }

      专用于合并事件（发射器的发射事件），从文档中可以看出，zip操作符可以合并两个发射器，最终合并出结果，且最终结果的数目只和结果集少的那个相同，
      从图一可知，结果集是以发送时最少的为主输出合并数量

<img width="614" alt="image" src="https://user-images.githubusercontent.com/67937122/162899152-ff1ef994-43a9-49b6-8e5b-e376430b4085.png">

图中可以看出，发射器A中的数据“5”在合并后抛弃了

    public static void zip(final TextView textView) {

            //getObservableThere()、getObservableTwo() 为第二篇文章中的工具方法，这一块在本文中省略
            Observable<String> observable = getObservableThere();
            Observable<String> observable2 = getObservableTwo();

            Observable.zip(observable, observable2, new BiFunction<String, String, JSONObject>() {
                @Override
                public JSONObject apply(@NonNull String response, @NonNull String response2) throws Exception {
                    //将两个发射器的结果合并在一块
                    return new JSONObject().put("one", response).put("two", response2);
                }
            }).subscribeOn(Schedulers.newThread())
                    .observeOn(AndroidSchedulers.mainThread())
                    .subscribe(new Consumer<JSONObject>() {
                        @Override
                        public void accept(@NonNull JSONObject jsonObject) throws Exception {
                            textView.setText(jsonObject.toString());
                        }
                    });
        }
        
<img width="667" alt="image" src="https://user-images.githubusercontent.com/67937122/162899254-3a449af1-26f6-4efd-9de8-ffd9e503f74a.png">

***-------------------------------------concat 顺序发出-----------------------------------------***

将两个发射器合并成一个发射器, 依次发送，发送完一个再接着发送第二个

      //原始方法
      public static <T> Observable<T> concat(ObservableSource<? extends T> source1, ObservableSource<? extends T> source2) {
          ObjectHelper.requireNonNull(source1, "source1 is null");
          ObjectHelper.requireNonNull(source2, "source2 is null");
          return concatArray(source1, source2);
      }
      
示例代码：     结果会依次输出 

      public static void concat(final TextView textView) {
              StringBuffer buffer = new StringBuffer();
              Observable.concat(Observable.just(1, 23, 465), Observable.just(789, 987, 666))
                      .subscribe(new Consumer<Integer>() {
                          @Override
                          public void accept(@NonNull Integer integer) throws Exception {
                              textView.setText(buffer.append(integer + "--->").toString());
                          }
               });
      }
      
      
***----------------------------------merge-交叉合并---------------------------------------***


      public static <T> Observable<T> merge(ObservableSource<? extends T> source1, ObservableSource<? extends T> source2) {
          ObjectHelper.requireNonNull(source1, "source1 is null");
          ObjectHelper.requireNonNull(source2, "source2 is null");
          return fromArray(source1, source2).flatMap((Function)Functions.identity(), false, 2);
      }
      
<img width="677" alt="image" src="https://user-images.githubusercontent.com/67937122/162899754-879660b1-4f97-4222-8b98-dcdfbb448451.png">

合并多个Observables的发射物， Merge 可能会让合并的Observables发射的数据交错，这也就是和concat的较大区别(挨个发送)，
由上图即可理解，不用等到 发射器 A 发送完所有的事件再进行发射器 B 的发送，他们是可以参杂交互发射事件的


       public static void merge(final TextView textView) {
              Observable.merge(Observable.just(1, 2, 7, 8), Observable.just(3, 4, 5)).subscribe(new Consumer<Integer>() {
                  @Override
                  public void accept(@NonNull Integer integer) throws Exception {
                      textView.append("merge :" + integer + "\n");
                  }
              });
          }
          
          
          
链接：https://blog.csdn.net/mwthe/article/details/82780193





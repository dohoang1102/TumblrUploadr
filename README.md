This iOS / Objective C code uploads multiple photos to Flickr as a slideshow / photoset.

Steps to get this to work:

Copy all files to a group in your project.  In your MyViewController.h, import the header and register the TumblrUploadrDelegate:

    #import "TumblrUploadr.h"

    @interface MyViewController : UIViewController <TumblrUploadrDelegate> {
    ...
    }

Now add your tumblr Consumer Key and Secret to the top of TumblrUploader.m!!  You'll also need an OAuth access token key and secret which you will add in the appropriate place below. (Getting the access token is beyond the scope of this project, for now...)

In your MyViewController.m implementation file, make a function to upload the files.  For now, you should thread the process as shown because otherwise the UI will lock up. This only works for iOS 4.

    - (void) uploadFiles {
        NSData *data = [NSData dataWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"tumb2" ofType:@"jpg"]];
        NSData *data2 = [NSData dataWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"tumblrpic" ofType:@"jpg"]];
        NSArray *array = [NSArray arrayWithObjects:data, data2, nil];
        dispatch_async( dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            TumblrUploadr *tu = [[TumblrUploadr alloc] initWithNSDataForPhotos:array andBlogName:@"supergreatblog.tumblr.com" andDelegate:self];
            dispatch_async( dispatch_get_main_queue(), ^{
                [tu signAndSendWithTokenKey:@"myAccessTokenKey" andSecret:@"myAccessTokenSecret"];
            });
        });
    }

Then add the two delegate methods, one of which should be called:

    - (void) tumblrUploadr:(TumblrUploadr *)tu didFailWithError:(NSError *)error {
        NSLog(@"connection failed with error %@",[error localizedDescription]);
        [tu release];
    }
    - (void) tumblrUploadrDidSucceed:(TumblrUploadr *)tu withResponse:(NSString *)response {
        NSLog(@"connection succeeded with response: %@", response);
        [tu release];
    }